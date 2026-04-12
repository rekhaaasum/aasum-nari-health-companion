# 🌸 AAsum Nari — Mythri Voice Health Companion

> *"Aap akeli nahin hain."* — You are not alone.

A voice-based AI health companion for South Asian women navigating perimenopause — built in Telugu, Urdu, and English, because health information should exist in the languages women actually speak.

**Live at [aasumnari.com](https://aasumnari.com)**

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

---

## What It Does

- 🎙️ **Speak, don't type** — voice in, voice out, no keyboard needed
- 🌐 **Three languages** — Telugu, Urdu, English with culturally appropriate honorifics (Meeru / Aap / You)
- 🏮 **Culturally grounded** — NFHS-5 data, Indian menopause age norms (46–48), South Asian home remedies (methi, flaxseeds, turmeric, ghee)
- 🌸 **Topic chips** — Sleep, Period, Mood, Heat, Why — pre-scripted spoken responses in all three languages for women who don't know where to start
- 💬 **Optional text display** — for accessibility or noisy environments
- 🐢 **Slow speech mode** — for easier comprehension
- 📱 **No app install** — opens in any mobile browser, works from a WhatsApp link
- 🩺 **Not a doctor** — always directs to medical care for diagnosis

---

## Who It's For

South Asian women aged 35–55, primarily in India and the diaspora, who are experiencing perimenopause symptoms and have nowhere to turn for information in their own language.

---

## Current Status

🟢 **Production Alpha — actively piloting with 10+ testers**

- Live at [aasumnari.com](https://aasumnari.com)
- Testers across Hyderabad, Chicago, and Redwood City
- Structured feedback via WhatsApp group
- Axiom logging capturing session data, language selection, latency, and device telemetry
- Iterating weekly based on real user feedback

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML/CSS/JS — no framework, runs in any browser |
| Hosting | GitHub Pages + custom domain (aasumnari.com) |
| AI | Gemini 2.5 Flash via Google AI Studio |
| Text-to-Speech | Google Cloud TTS — Telugu, Urdu, English voices |
| Speech Input | Web Speech API (browser-native) |
| Backend Proxy | Vercel serverless function (private repo: aasum-nari-service) |
| Logging | Axiom — structured session and performance logs |
| VAD | 2-second silence threshold, 60-second max speech |

---

## Architecture

```
User speaks
    ↓
Web Speech API (browser)
    ↓
Vercel proxy (api/chat.js)
    ↓
Gemini 2.5 Flash → reply text
    ↓
Google Cloud TTS → audio
    ↓
Mythri speaks back
    ↓
Axiom logs the session
```

The frontend and backend are intentionally separated — the Vercel proxy protects all API keys and handles CORS for both `aasumnari.com` and `rekhaaasum.github.io`.

---

## Axiom Log Schema

Every conversation turn logs:

```json
{
  "user_phone": "...",
  "session_id": "session_timestamp_random",
  "language_selected": "te/ur/en",
  "reply_language": "te/ur/en",
  "device_os": "iOS/Android/Windows/MacOS",
  "device_browser": "Safari/Chrome/Edge",
  "latency_ms": 843,
  "user_speech_duration_ms": 4200,
  "type": "chat/tts",
  "status": "success/error"
}
```

No transcripts are logged. Privacy by design.

---

## Why I Built This

I run community health programs for South Asian women. Again and again, I saw women who had been suffering for years without knowing what was happening to their bodies. Perimenopause is not taught, not discussed, not named.

Mythri is the scalable version of what I do in person — reaching women I can't reach through library talks or panels. One conversation at a time, in their language.

---

## What's Next

- [ ] Gemini Live migration — real-time voice, interruptions, natural conversation
- [ ] iOS audio improvements
- [ ] Expand to Hindi
- [ ] Symptom tracker across sessions
- [ ] Partner with women's health NGOs in Andhra Pradesh and Telangana
- [ ] Ingredients scanner — scan food labels for perimenopause-relevant nutrients

---

## About the Builder

Built by **Rekha** — a South Asian woman, community health advocate, and AI product builder with 11+ years of enterprise technology experience. This project sits at the intersection of two things that matter: closing health equity gaps for South Asian women, and building AI tools that solve real problems for real people.

This is not a portfolio piece. It's a mission that needed a product.

---

## Development Workflow

### Stack Notes

- Vercel backend must use `module.exports` (CommonJS), not `export default`
- Language switching uses in-app state, not `window.location.reload()` — reloading breaks mic permissions on Android and iOS
- Phone entry screen is shown by default in HTML, hidden via JS when user is registered — ensures `user_phone` is never `unknown` in logs
- CORS must explicitly whitelist both `aasumnari.com` and `rekhaaasum.github.io`

### Linting (ESLint + Husky)

```bash
npm install
npm run lint
npm run lint:fix
```

Every `git commit` runs ESLint automatically via Husky. Commits blocked on Error-level issues only — style preferences not enforced.

---

*Mythri is not a medical device and does not provide medical advice. She is an informational and emotional support companion. Always consult a qualified healthcare professional for diagnosis and treatment.*
