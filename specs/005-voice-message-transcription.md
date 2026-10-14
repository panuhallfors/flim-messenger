---
id: 005
slug: voice-message-transcription
title: Voice message transcription
status: proposed
created: 2026-10-13
amends: [004]
platforms: [ios, android]
---

# 005 — Voice message transcription

## Intent

The user chooses how Flim transcribes voice messages, from a section on the main settings screen. They pick Flim's service, a server of their own, or this device, and finish that method's setup before it is used. Flim's setup creates an account with an email and a code, then a promo code (in-app subscription is shown but not available yet).

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] Settings has a Voice message transcription section on the main settings screen.
- [ ] The user picks a method first. That method is used only after its setup is finished.
- [ ] Flim's setup takes an email, then a code sent to that email, then a promo code for a plan. A subscription option is visible but disabled, with a note that it is coming later. Afterwards the user can see the account and the plan.
- [ ] A custom server's setup explains, in everyday language, that any OpenAI-compatible service works — for example OpenAI, Cortecs (a European provider), or OpenRouter — and briefly where to copy the key and the address. The user then enters the addresses, key(s), and the models. Saving runs a short check against that server first; only a successful check makes it the method in use. A failed check keeps the method unset (or the previous working setup) and shows the same everyday failure wording used on voice-message briefs. When the summary uses a different address, a second key field appears (defaulting to the same key).
- [ ] On this device can be turned on when the device is able to transcribe and summarise there. The user sees, before turning it on, that the quality is lower than an external service. A device that cannot do it shows the choice and why it cannot be turned on.

## Non-goals

- The hosted Flim transcription service (the listener, its prompts, its hosting). Its own decision.
- How a chosen method turns audio into a title, summary, and transcript. Each listener is its own decision. Custom server: decision 006. On this device: decision 007.
- Changing decision 004's race, wait, room notice, or who may publish.
- Using the Matrix account as the Flim transcription account.

## Notes

Decision 004 owns the brief on the voice message, the room notice, and the failure statuses. This decision adds the choice of method.

New user-facing strings belong to the Flim catalog (decision 002).

**Disclosure (store / privacy):** Before a method is in use, tell the user clearly where the voice message and room context go — Flim’s third-party ZDR EU service, a third-party server they name, or only this device. Custom-server options (OpenAI, Cortecs, OpenRouter, or any other OpenAI-compatible host) are services the user is free to obtain and use themselves; the app does not hide that the audio leaves the phone for that host. Selector and setup copy own this; a later decision may add an explicit consent step or Flim service terms if store review requires more than that copy.

Design (replay-canonical with this decision): [`005-voice-message-transcription.design.md`](005-voice-message-transcription.design.md)

## Verification

### ios

- Tests: none yet (the selector, each setup step, and the unavailable on-device choice are lockable with unit tests and previews)
- Agent UI: none

### android

- Tests: none yet (the selector, each setup step, and the unavailable on-device choice are lockable with unit tests and previews)
- Agent UI: none

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-13: proposed
- 2026-10-13: written as the finished setup — selector, then Flim account and plan (promo or subscription), custom server, on-device when the device can. No interim wiring in this decision
- 2026-10-13: failure statuses live in decision 004; this decision does not restate them
- 2026-10-13: custom-server listener is decision 006
- 2026-10-13: on-device listener, and the test behind "the device can", are decision 007
- 2026-10-13: Android implements the same three choices as iOS (Flim, custom server, on this device)
- 2026-10-13: custom-server setup explains how to get a key and an address from a service such as OpenAI, Cortex, or OpenRouter
- 2026-10-13: Cortecs (cortecs.ai) is the European example; any OpenAI-compatible service works
- 2026-10-14: plan step — promo works; in-app subscription shown disabled with coming-later copy
- 2026-10-14: platforms = ios, android
- 2026-10-14: second key when summary address differs (default same key)
- 2026-10-14: amended by 009 — not-set-up bubble opens setup, not Try again
- 2026-10-14: Notes — clear disclosure that audio/context go to a third party or stay on device; custom hosts are services the user may use themselves
- 2026-10-14: custom-server save runs a bundled-clip check (via 006) before the method is in use; failures use 004 wording
