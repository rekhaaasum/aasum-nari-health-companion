# Mythri Frontend Fixes (target: code freeze Oct 7, demo Oct 10, 2026)

Small, low-risk fixes to alpha-testing issues before the Oct 10 workshop demo. Rules are in `CLAUDE.md`.

## Priority tiers

Work in this order: **F0** (Tier 1, do first, it blocks Android users today), then **F5** (Tier 1, small privacy fix), then **F2** (Tier 1, must), then **F1** (Tier 2), then **F4** (Tier 2, needs Ray's approved text), then **F3** (Tier 2, the first to drop if time runs short). The `BE_PLAN.md` (backend repo) holds the cut line and the venue tests; the same Saturday cut line applies here.

## Task F0 (Tier 1, do first): Android gets stuck in listening mode and repeats words

**Reported Oct 1 by a tester on Android Chrome:** the greeting plays, he speaks, the orb stays in listening mode and Mythri never replies. The on-screen transcript shows growing repeats: "hi hi hi my hi my tree hi my tree how hi my tree how are you...".

**Root cause, from reading `startListening` (confirm before changing):**
1. **Stuck in listening.** Android Chrome ends a recognition session on its own after a short pause. `recognition.onend` then clears the pending 3-second `silenceTimer`, and because `accumulatedFinal` already has text, it neither restarts nor processes the speech. The orb stays in `listening` forever. The 60-second cap uses the same `silenceTimer` variable, so it is cleared too, and the 20-second fallback only applies when nothing was heard. Desktop Chrome keeps the session open, so the silence timer fires there, which is why this shows up on Android.
2. **Repeated words.** On Android, each new final result contains the whole phrase so far, and `onresult` appends every one of them to `accumulatedFinal`. The backend's `normalizeTranscript` only removes exact repeats, not these growing ones.

**Changes**
- In `recognition.onend`: if `state === 'listening'` and `accumulatedFinal.trim()` has text, call `processUserSpeech(accumulatedFinal.trim(), <duration since speechStartTime>)`. `processUserSpeech` sets the state to `thinking` straight away, so the silence-timer path and `onend` cannot both process the same turn. Keep the existing restart and idle behavior when no text was heard.
- In `recognition.onresult`, when a final result arrives: if it starts with the current `accumulatedFinal` text (ignoring case and surrounding spaces), replace `accumulatedFinal` with it instead of appending. Otherwise append as today.
- On Android only (not iOS, not desktop), set `recognition.continuous = false`, so Android ends each turn when she stops speaking and the fixed `onend` sends it. Leave desktop and iOS settings unchanged.
- Keep a safety net: if the orb has been in `listening` for 15 seconds after speech was heard with no reply started, process whatever text exists.
- Bump `APP_VERSION`.

**Manual test for Ray (and the tester who reported it):** on Android Chrome, in English, say "Hi, how are you doing?" and stop. Mythri should reply within a few seconds, and the transcript should read once, without repeats. Do 10 turns in a row. Then repeat once on a desktop browser and once on an iPhone to check nothing regressed.

## Task F1 (Tier 2): Let Urdu speakers actually speak Urdu

**Problem:** in `startListening` (the `langMap` near `recognition.lang`), Urdu is hard-coded to `en-IN` on every device. Urdu speech is transcribed as English guesses, so Urdu voice input is effectively broken. This is the alpha issue "Urdu speech recognition falls back to en-IN on all devices". It is the code, not the browser.

**Changes**
- Non-iOS devices: map `ur` to `ur-IN`. Leave the iOS mapping unchanged for now.
- In `recognition.onerror`, handle `language-not-supported`: if the current recognition language is not `en-IN`, set a session-level flag, show a toast from a new `LANGS` string `voiceLangFallback` (English: "Urdu voice input isn't available on this device yet. You can speak in English, or type instead."), and restart recognition with `en-IN`. While the flag is set, use `en-IN` for Urdu for the rest of the session. Apply the same fallback to Telugu so a device without Telugu support also degrades gracefully.
- Bump `APP_VERSION` to `2026-10-03-v16`.

**Manual test for Ray:** on Android Chrome (Kamakshi's Samsung is ideal), select Urdu, say a symptom in Urdu, and turn on text display. The transcript should appear in Urdu script. Then do the same in Telugu to check nothing regressed.

## Task F5 (Tier 1, right after F0): Stop sending phone numbers saved by the old registration screen

**Problem:** the phone-number screen is switched off, but testers who registered before still have their number saved in the browser (`localStorage` key `mythri_user_phone`) and in old links (`?u=<number>`). On every visit the app reads it back, Force URL Sync copies it into the address bar, and it is sent as `user_phone` with every request, so real phone numbers keep reaching Axiom.

**Changes**
- Where `USER_PHONE` is first set from `urlPhone` or `localStorage`, check the value. If it looks like a phone number (only digits, optionally a leading `+`, 7 or more digits), discard it: generate a new guest ID the same way the guest bypass does, save that to `mythri_user_phone`, and use it as `USER_PHONE`.
- Do this before the Force URL Sync check, so the redirect bakes the guest ID into the address bar instead of the number.
- Tester codes like `t07` and existing `guest_` and `anon_` IDs are left alone.
- Do not remove the dormant `savePhone` screen code in this task (that is cleanup for after Oct 10).
- Keep the known solved fixes in `CLAUDE.md` working: Force URL Sync, `setLang` preserving `?u=`, the WebView banner.

**Manual test for Ray:** open the app with `?u=5551234567`. The address bar should switch to a `guest_` ID, and the next Axiom entries should show that guest ID, not the number. Then open with `?u=t07` and check it stays `t07`, including after switching language.

## Task F2 (Tier 1): Stop failing silently

**Problem:** `recognition.onerror` only shows a message for `not-allowed`. Every other error (no speech heard, network drop, no microphone) silently returns the orb to idle, so the user can't tell what happened. This matters at a venue with unreliable Wi-Fi.

**Changes:** in `recognition.onerror`, show a toast using the strings that already exist in `LANGS` (already translated, so no new translations are needed):
- `no-speech` → `noSpeech`
- `network` → `errorAPI`
- `audio-capture` → `errorMic`
- `not-allowed` → `errorMic` (unchanged)
- `aborted` → no toast (this happens when the app itself stops recognition)

Keep the existing reset to idle.

**Manual test for Ray:** on Android Chrome, tap the orb and stay silent (expect the "didn't catch that" message); turn on airplane mode and tap (expect the connection message).

## Task F3 (Tier 2, drop first if behind): Device details and troubleshooting events

Needs backend Task 10 deployed to the same preview. Goal: when a tester says "it didn't work on my phone," the logs show which phone, which browser, and where it failed.

**1. Fix browser detection in `detectDevice`.** Samsung Internet, Mi Browser and UC Browser all include "Chrome" in their user agent, so today they are logged as Chrome. These browsers are common in India and handle speech recognition differently. Before the Chrome check, detect `SamsungBrowser` → `SamsungInternet`, `MiuiBrowser` → `MiBrowser`, `UCBrowser` → `UCBrowser`, and `FBAN`/`FBAV`/`Instagram` → `InAppBrowser`. Keep the existing WebView check first.

**2. Collect `device_details` once at page load** and send it with every `chat`, `tts` and `event` request:
- `os_version` and `device_model`: from `navigator.userAgentData.getHighEntropyValues(['platformVersion', 'model'])` where available (Chrome on Android). Chrome hides the phone model in the ordinary user agent, so this is the only way to get it. Fall back to parsing the user agent. Must not block the first request by more than 300 ms.
- `browser_version`: major version number.
- `network_type`: `navigator.connection.effectiveType` if available (for example `4g`, `3g`).
- `device_memory_gb`: `navigator.deviceMemory`; `cpu_cores`: `navigator.hardwareConcurrency`. These flag low-end phones.
- `speech_api`: whether `SpeechRecognition` or `webkitSpeechRecognition` exists.
- `in_app_browser`: true for Android WebView or the in-app browsers above.
- `debug_mode`: true when the URL has `debug=1`.

**3. Send small events** with `fetch(..., { keepalive: true })` and `type: "event"`, never blocking the user and ignoring failures:
- `page_load` once, sent only after the Force URL Sync redirect check has passed, so the redirect does not log two page loads.
- `mic_permission` with `permission_state` from `navigator.permissions.query({ name: 'microphone' })` where supported.
- `stt_start` with `stt_lang`; `stt_result` with `time_to_first_result_ms` on the first result of a turn; `stt_error` with `error_code` from `recognition.onerror`; `stt_restart` with `restart_count` in `recognition.onend` when it restarts; `stt_lang_fallback` from Task F1.
- `audio_play_blocked` when `play()` is rejected (the tap-to-hear path); `audio_onended_fallback` when the duration timer resets instead of `onended`; `audio_error` from `audio.onerror`.
- `tts_request_error` and `chat_request_error` with `http_status` when a backend call fails.
- Never include transcript or reply text in any event.

**4. Show a session code for testers.** When the URL has `debug=1`, show a small line at the bottom of the screen: "Session: " plus the last 6 characters of `SESSION_ID`. A tester can read it out, and Ray can find that session in Axiom.

**Manual test for Ray:** open the preview on an Android phone with `?debug=1`, run one conversation, deny the microphone once, and check in Axiom that events arrive with the phone model, browser and network type filled in.

## Task F4 (Tier 2): "Before we talk" agreement screen

**Why:** Mythri should tell every woman, once and clearly, what Mythri is and is not, what happens to what she says, and where to get urgent help, and record that she agreed. The approved AAsum Nari medical disclaimer stays word for word.

**Changes**
- On first visit (and again whenever `CONSENT_VERSION` changes), show a full-screen card in the selected language before the orb. Use new `LANGS` strings. English text, which Ray must approve before merge:
  - Title: "Before we talk"
  - "Mythri is a friend for information and support. She is not a doctor and cannot diagnose or treat you."
  - "Mythri is not for emergencies. In India, call 112 for an emergency, or Tele-MANAS on 14416 for free emotional support, day or night. Outside India, call your local emergency number."
  - "When you speak, your voice is turned into text by your browser's speech service, and Google's AI service writes Mythri's reply. AAsum Nari does not store recordings or what you say. We keep basic technical details, like your phone model, to fix problems."
  - The existing approved disclaimer text from `LANGS`.
  - Button: "I understand and agree". Links to "Terms of use" and "Privacy policy" only if `TERMS_URL` and `PRIVACY_URL` constants are set (leave them empty for now; hide the links when empty).
- On agree: save `mythri_consent` = `{ version: CONSENT_VERSION, at: <ISO time> }` in `localStorage` and send a `consent_given` event with `consent_version` (backend Task 10 accepts it). Start `CONSENT_VERSION` at `"1"`.
- If `localStorage` is unavailable, show the card every visit rather than skip it.
- The card must not interfere with the Force URL Sync redirect or `setLang` reload (see `CLAUDE.md`). Switching language on the card should show the card in the new language.
- `te` and `ur` strings are `TODO(Ray): translate`.

**Manual test for Ray:** open the app in a private window on Android Chrome: the card appears; agree; reload: it does not appear again; switch language on the card: it shows in that language.

## Tester links (no code needed)

Do not bring back the phone number screen. Give each tester a personal link that carries a short tester code instead of a phone number, for example `...aasum_nari_voice.html?u=t07&debug=1`. The app already accepts any value in `u=`, keeps it across language switches, and stores it on the phone. Keep the list of which code belongs to which tester in your own private sheet, not in the repo or the logs. Opening the new link also replaces any phone number an earlier link saved on that tester's phone.

Tell testers to open the link in Chrome, not inside WhatsApp. WhatsApp's built-in browser blocks the microphone, and the app shows a banner saying so.

## iPhone issues: no code changes before the demo

The open iOS issues (autoplay blocked with a tap-to-hear workaround, unreliable `onended` with a duration fallback already in place, and the every-other-tap problem) stay as they are until after Oct 10. Changing audio handling a week before a live demo risks breaking what works.

- **Demo on Android Chrome**, not an iPhone.
- If an attendee uses an iPhone, tell them to tap "tap to hear" when it appears.
- On Oct 6, ask Kamakshi to try to reproduce the every-other-tap problem and write down exact steps. If she can, fix it after Oct 10 with those steps.

## Schedule

| Date | Work |
|---|---|
| Thu Oct 1 | **F0**, tested by Ray and the tester who reported it before anything else. Then **F5** |
| Fri Oct 2 | F2, then F1 |
| Sat Oct 3 | F4 (once Ray approves the English text), then F3 if backend Task 10 is on the preview. Cut line at end of day |
| Sun Oct 4 | Ray supplies Telugu and Urdu for `voiceLangFallback` and the consent card, and sends tester links |
| Mon Oct 5 | Deploy with the backend changes; testers use their tester links on their own phones |
| Tue Oct 6 | Ray reviews Axiom by tester code and device; fix what shows up |
| Wed Oct 7 | Code freeze |
| Thu Oct 8 | Venue test round 2 (go/no-go), see `BE_PLAN.md` (backend repo) |

## After Oct 10

- iOS audio issues (autoplay, `onended`, every-other-tap).
- Remove the dormant phone-number registration screen and `savePhone` code (F5 already stops saved numbers from being sent).
