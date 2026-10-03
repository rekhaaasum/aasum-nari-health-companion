# aasum-nari-health-companion: instructions for Claude Code

Public frontend for Mythri, AAsum Nari's voice health companion. The app is one page, `aasum_nari_voice.html`, served by GitHub Pages. It records speech with the browser's Web Speech API, sends text to a private backend, and plays back the audio it returns. The current work plan is in `FE_PLAN.md`.

## Hard constraints

1. Do not change the request payloads sent to the backend or the fields read from its responses, except for the additions `FE_PLAN.md` describes (`device_details` and `type: "event"` requests). Telemetry never includes what the user said or what Mythri replied.
2. Do not add libraries, CDNs, build steps or frameworks. Keep it one self-contained HTML file.
3. Never add console logging of what the user said or what Mythri replied.
4. Do not change `index.html` (the homepage) unless a task says so.
5. This repo is public. Never commit secrets, keys, phone numbers or backend-internal details.
6. Keep the existing `LANGS` pattern for all user-facing text: every new string gets `en`, `te` and `ur` entries. Put English in all three and mark `te` and `ur` with `// TODO(Ray): translate`. Never write Telugu or Urdu yourself.

## Known solved issues (fixed in v15 and v15.1, do not re-investigate or undo)

| Issue | Root cause | Fix in the code |
|---|---|---|
| WhatsApp WebView strips `?u=` URL params | WebView URL handling | Force URL Sync redirect |
| `setLang()` drops `?u=` | Language switch reloaded without params | Params preserved on reload |
| Android WebView blocks the microphone | WebView security policy | Chrome redirect banner (`webview-banner`) |
| Language leaking across sessions | Session state not cleared | `window.location.reload()` on language switch |
| Android Chrome stuck in listening mode and repeated words in the transcript (F0, v15.1) | Android ends the recognition session by itself, and each final result repeats the whole phrase so far | In `startListening`: `onend` sends the heard text; a final result that extends `accumulatedFinal` replaces it instead of appending; `continuous = false` on Android only; 15-second safety net; all sends go through `sendTurn` |

Any change near these (URL params, `setLang`, `detectDevice`, the WebView banner, `startListening` and its `onend`/`onresult` handlers) must keep them working. Mention in the pull request how you checked.

## How to work

- Work through `FE_PLAN.md` one task at a time, in the tier order at the top of the plan, not in task-number order. One branch and one pull request per task.
- Before editing, restate the task and list what you will change.
- You cannot test a microphone or a phone. End every task with a short manual test checklist for Ray, naming the device and browser to use.
- Stop after each task and wait for Ray's go-ahead. Never merge to `main` yourself.
- If `FE_PLAN.md` does not match the code, stop and ask instead of guessing.
