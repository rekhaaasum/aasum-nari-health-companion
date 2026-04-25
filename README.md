# 🌸 AAsum Nari — Mythri Voice Health Companion

> 🌸 **Looking for our mission and community stories? Visit [aasumnari.org](https://www.aasumnari.org/)**

> *"Aap akeli nahin hain."* — You are not alone.

A voice-based AI health companion for South Asian women navigating perimenopause — built in Telugu, Urdu, and English, because health information should exist in the languages women actually speak.

**Live at [Mythri — AAsum Nari Voice Companion](https://aasumnari.com/aasum_nari_voice.html)**

---

## The Problem

Perimenopause affects every woman. But for millions of South Asian women, it happens in silence.

- Doctors dismiss symptoms as "stress" or "just getting older"
- Families don't talk about it
- Health information in Telugu or Urdu is nearly nonexistent
- Women suffer through hot flashes, sleepless nights, mood changes, and joint pain — alone, and without language for what they're experiencing

AAsum Nari exists to close that gap.

---

## Meet Mythri

Mythri (మైత్రి) means friendship. She is the voice companion inside AAsum Nari.

A woman taps the flower, speaks in her language, and Mythri responds — warm, spoken, culturally informed, and free of judgment. Like a knowledgeable friend who happens to know a lot about women's health.

Mythri follows the **Actionable Empathy framework** in every response:
1. **Empathy** — validate the feeling warmly
2. **Environment** — one practical environment tip
3. **Nutrition** — one South Asian-grounded nutrition tip
4. **Movement/Breath** — one gentle movement or breathing tip
5. **The Science** — one warm sentence explaining the biology behind the symptom

---

## What It Does

- 🎙️ **Speak, don't type** — voice in, voice out, no keyboard needed
- 🌐 **Three languages** — Telugu, Urdu, English with culturally appropriate honorifics (Meeru / Aap / You)
- 🧠 **Symptom-aware brain** — detects which symptom cluster the user is describing and routes to the correct tips. Hot flash tips never appear for joint pain.
- 🏮 **Culturally grounded** — NFHS-5 data, Indian menopause age norms (46–48), South Asian home remedies (methi, flaxseeds, turmeric, ghee, sesame, ragi)
- 🌸 **Topic chips** — Sleep, Period, Mood, Heat, Why — pre-scripted spoken responses in all three languages for women who don't know where to start
- 💬 **Optional text display** — for accessibility or noisy environments
- 🐢 **Slow speech mode** — for easier comprehension
- 📱 **No app install** — opens in any mobile browser, works from a WhatsApp link
- 🩺 **Not a doctor** — directs to medical care when symptoms are severe, new, or worsening

---

## Mythri's Brain — Symptom-to-Pillar Matrix

Mythri uses a server-side keyword detection engine to identify which symptom cluster the user is describing before calling Gemini. Each cluster maps to specific, hard-coded tips — eliminating generic responses and hallucination risk.

| Cluster | Symptoms Detected | Example Tips |
|---|---|---|
| **Vasomotor** | Hot flashes, night sweats, flushing, heat | Cooling breath (Sitali), cotton layers, avoid spicy masalas |
| **Metabolic** | Fatigue, brain fog, weight, memory, focus | Lentils & chickpeas, resistance training, healthy snacks |
| **Skeletal** | Joint pain, bone ache, stiffness, back/knee | Sesame & almonds, 1200mg calcium, quadriceps exercises |
| **Emotional** | Mood, anxiety, sleep, insomnia, stress | Box breathing (4-4-4-4), omega-3 seeds, sleep hygiene |
| **Urogenital** | Irregular periods, bleeding, vaginal dryness | Iron-rich foods, pelvic tilts, safety kit |

Keyword detection works across English, Telugu script, and Urdu script. The `why` field in each cluster gives Mythri the biological explanation (e.g. "Estrogen affects the brain's thermostat") to explain in simple, warm language.

---

## Who It's For

South Asian women aged 35–55, primarily in India and the diaspora, who are experiencing perimenopause symptoms and have nowhere to turn for information in their own language.

---

## Current Status

🟢 **Production Alpha — v15 stability release, actively piloting with testers**

- Live at [aasumnari.com/aasum_nari_voice.html](https://aasumnari.com/aasum_nari_voice.html)
- Testers across Hyderabad, Chicago, and Redwood City
- Structured feedback via WhatsApp group ("AAsum Nari Health Circle")
- Multi-region routing: US East (`iad1`) + Singapore (`sin1`) for India traffic
- Axiom logging capturing session data, language, latency, symptom clusters, and device telemetry
- Frontend and backend versioning for deployment traceability
- Iterating weekly based on real user feedback

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML/CSS/JS — no framework, runs in any browser |
| Hosting | GitHub Pages + custom domain (aasumnari.com) |
| AI | Gemini 2.5 Flash via Google AI Studio |
| Text-to-Speech | Google Cloud TTS — Telugu, Urdu, English voices |
| Speech Input | Web Speech API (browser-native) with VAD |
| Backend Proxy | Vercel Pro serverless (private repo: aasum-nari-service) |
| Logging | Axiom — structured session and performance logs |
| VAD | 3-second silence threshold, 60-second max speech |
| Regions | `iad1` (Washington D.C.) + `sin1` (Singapore) |

---

## Architecture

```
User speaks
    ↓
Web Speech API (browser VAD — 3s silence threshold)
    ↓
Vercel proxy (api/chat.js) — immediate "received" log to Axiom
    ↓
Transcript normalization — strips Android STT duplicates
    ↓
Symptom cluster detection — keyword match across EN/TE/UR
    ↓
Gemini 2.5 Flash + Pillar tips injected into system prompt
    ↓
Google Cloud TTS → MP3 audio
    ↓
Mythri speaks back
    ↓
Axiom finally log — fires even on timeout or crash
```

The frontend and backend are intentionally separated — the Vercel proxy protects all API keys and handles CORS for both `aasumnari.com` and `rekhaaasum.github.io`.

---

## Axiom Log Schema

Every conversation turn logs two entries — an immediate `received` log on arrival, and a `success/error` log in the `finally` block. This ensures India sessions are captured even if the function times out.

```json
{
  "backend_version": "chat-15-stability-observability",
  "frontend_version": "2026-04-23-v15",
  "user_phone": "...",
  "session_id": "session_timestamp_random",
  "session_source": "frontend | backend_generated",
  "language_selected": "te/ur/en",
  "symptom_cluster": "Vasomotor | Metabolic | Skeletal | Emotional | Urogenital | unknown",
  "device_os": "iOS/Android/Windows/MacOS",
  "device_browser": "Safari/Chrome/AndroidWebView",
  "latency_ms": 843,
  "total_duration_ms": 1420,
  "user_speech_duration_ms": 4200,
  "region": "iad1 | sin1",
  "type": "chat/tts",
  "status": "received | success | error",
  "error_type": "gemini_timeout | system_error"
}
```

No transcripts are logged. Privacy by design.

---

## Stability & Resilience (v15)

- **Graceful timeout fallback** — on Gemini timeout, backend returns `200` with a warm spoken fallback ("I heard you, but my connection is slow — say that in one short sentence"). User hears Mythri instead of hitting a silent tap loop.
- **AbortSignal timeout** — 55-second hard cutoff on both Gemini and TTS calls, within Vercel Pro's 60-second function limit.
- **Android WebView detection** — shows a banner directing Android users to open in Chrome when WhatsApp WebView is detected (mic blocked in WebView).
- **Transcript normalization** — strips Android STT cumulative duplicates ("my joints hurt my joints hurt") before sending to Gemini.
- **Force URL Sync** — if phone is in localStorage but not in URL (WhatsApp strips params), redirects once to bake `?u=` into the URL.
- **Language switch identity preservation** — `setLang()` always carries `?u=` through language switches so identity is never lost mid-session.

---

## Why I Built This

I run community health programs for South Asian women. Again and again, I saw women who had been suffering for years without knowing what was happening to their bodies. Perimenopause is not taught, not discussed, not named.

Mythri is the scalable version of what I do in person — reaching women I can't reach through library talks or panels. One conversation at a time, in their language.

---

## What's Next

- [ ] Gemini Live migration — real-time voice, interruptions, natural conversation
- [ ] Vertex AI migration for enterprise-grade reliability
- [ ] Community stories integration — 100 real perimenopause stories from Project 100 India pilot
- [ ] Hindi language support
- [ ] Symptom tracker across sessions
- [ ] Phone number hashing for enhanced privacy (SHA-256)
- [ ] Partner with women's health NGOs in Andhra Pradesh and Telangana
- [ ] Google Cloud for Startups ($350K AI-First track) application

---

## About the Builder

Built by **Rekha** — a South Asian woman, community health advocate, and AI product builder with 18+ years of enterprise technology experience across fintech, insurance, and healthcare IT. This project sits at the intersection of two things that matter: closing health equity gaps for South Asian women, and building AI tools that solve real problems for real people.

This is not a portfolio piece. It's a mission that needed a product.

---

## Development Workflow

### Stack Notes

- Vercel backend uses ES module `export default` syntax (configured in `vercel.json`)
- Language switching preserves `?u=` param — never uses `window.location.reload()` which breaks mic permissions
- Phone entry screen shown by default, hidden via JS when user is registered — ensures identity is never `unknown` in logs
- CORS explicitly whitelists `aasumnari.com`, `www.aasumnari.com`, and `rekhaaasum.github.io`
- Android WebView (WhatsApp) detected via `wv)` flag in user agent — mic blocked, banner shown
- `session_source: backend_generated` in logs signals frontend state failure

### Versioning

- `APP_VERSION` constant in HTML (e.g. `2026-04-23-v15`) sent in every API request
- `BACKEND_VERSION` constant in `chat.js` logged to Axiom on every call
- Both visible in Axiom logs — tells you exactly which deploy each session ran on

### Linting (ESLint + Husky)

```bash
npm install
npm run lint
npm run lint:fix
```

Every `git commit` runs ESLint automatically via Husky. Commits blocked on Error-level issues only.

---

*Mythri is not a medical device and does not provide medical advice. She is an informational and emotional support companion. Always consult a qualified healthcare professional for diagnosis and treatment.*
