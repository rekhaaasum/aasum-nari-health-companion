# 🌸 AAsum Nari — Voice Health Companion

> *"Aap akeli nahin hain."* — You are not alone.

A voice-based AI health companion for South Asian women navigating perimenopause — built in Telugu, Urdu, and English, because health information should exist in the languages women actually speak.

---

## The Problem

Perimenopause affects every woman. But for millions of South Asian women in India, it happens in silence.

- Doctors dismiss symptoms as "stress" or "just getting older"
- Families don't talk about it
- Health information in Telugu or Urdu is nearly nonexistent
- Women suffer through hot flashes, sleepless nights, mood changes, and joint pain — alone, and without language for what they're experiencing

AAsum Nari exists to close that gap.

---

## What It Does

AAsum Nari is a voice-first conversational AI companion. A woman taps a button, speaks in her language, and receives a warm, spoken response — culturally informed, practically useful, and free of judgment.

- 🎙️ **Speak, don't type** — voice in, voice out
- 🌐 **Three languages** — Telugu, Urdu, English
- 🏮 **Culturally grounded** — references familiar foods, Ayurvedic framing, family dynamics
- 💬 **Optional text display** — for accessibility or noisy environments
- 🐢 **Slow speech mode** — for easier comprehension
- 🩺 **Not a doctor** — always directs to medical care for diagnosis

---

## Who It's For

South Asian women — primarily in India — aged 35–55 who are experiencing perimenopause symptoms and have nowhere to turn for information in their own language.

**First users:** Physical therapists in Hyderabad, India, piloting this with their patients.

---

## Why I Built This

I run community health programs for South Asian women in the US. Again and again, I saw women who had been suffering for years without knowing what was happening to their bodies. Perimenopause is not taught, not discussed, not named.

AAsum Nari is the scalable version of what I do in person — reaching women I can't reach through library talks or panels. One conversation at a time, in their language.

---

## Tech Stack

- **Vanilla HTML/CSS/JS** — no framework, runs in any browser
- **Web Speech API** — browser-native voice recognition (no third-party dependency)
- **Web Speech Synthesis** — spoken responses with multilingual voice support
- **Anthropic Claude API** — culturally-informed, conversationally-tuned AI responses
- **GitHub Pages** — free deployment, shareable link, no app install needed

---

## How to Run It

1. Clone or download this repo
2. Open `aasum_nari_voice.html` in a text editor
3. Add your Anthropic API key:
```js
headers: {
  'Content-Type': 'application/json',
  'x-api-key': 'your-key-here',
  'anthropic-version': '2023-06-01'
}
```
4. Serve locally (required for mic access):
```bash
python3 -m http.server 8080
```
5. Open `http://localhost:8080/aasum_nari_voice.html` in Chrome

Or deploy to GitHub Pages for a shareable `https://` link.

---

## Status

🟡 **Prototype — actively piloting**

Currently being tested with physical therapists in Hyderabad, India as a patient-facing tool. Gathering feedback on language accuracy, cultural resonance, and clinical usefulness.

---

## What's Next

- [ ] Add Hindi language support
- [ ] Symptom tracker across sessions
- [ ] Referral to local gynecologists / telemedicine
- [ ] Offline mode for low-connectivity areas
- [ ] Partner with women's health NGOs in Andhra Pradesh and Telangana

---

## About the Builder

Built by **Rekha** — a South Asian woman, community health advocate, and emerging AI product builder. This project sits at the intersection of two things I care about: closing health equity gaps for South Asian women, and building AI tools that solve real problems for real people.

This is not a portfolio piece for its own sake. It's a mission that needed a product.

---

*AAsum Nari is not a medical device and does not provide medical advice. It is an informational and emotional support tool. Always consult a qualified healthcare professional for diagnosis and treatment.*
