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

The user chooses how Flim transcribes voice messages, from a section on the main settings screen. They pick Flim's service, a server of their own, or this device, and finish that method's setup before it is used. Flim's setup creates an account with an email and a code, then a promo code or a subscription.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] Settings has a Voice message transcription section on the main settings screen.
- [ ] The user picks a method first. That method is used only after its setup is finished.
- [ ] Flim's setup takes an email, then a code sent to that email, then either a promo code or a subscription. Afterwards the user can see the account and the plan.
- [ ] A custom server's setup takes the addresses, a key, and the models that transcribe and summarise, and then that server is the method in use.
- [ ] On this device can be turned on when the device is able to transcribe and summarise there. The user sees, before turning it on, that the quality is lower than an external service. A device that cannot do it shows the choice and why it cannot be turned on.

## Non-goals

- The hosted Flim transcription service (the listener, its prompts, its hosting). Its own decision.
- How a chosen method turns audio into a title, summary, and transcript. Each listener is its own decision. Custom server: decision 006. On this device: decision 007.
- Changing decision 004's race, wait, room notice, or who may publish.
- Using the Matrix account as the Flim transcription account.

## Notes

Decision 004 owns the brief on the voice message, the room notice, and the failure statuses. This decision adds the choice of method.

New user-facing strings belong to the Flim catalog (decision 002).

Design (replay-canonical with this decision): [`005-voice-message-transcription.design.md`](005-voice-message-transcription.design.md)

## Verification

### ios

- Tests: none yet (the selector, each setup step, and the unavailable on-device choice are lockable with unit tests and previews)
- Agent UI: none

### android

- Tests: none yet (the selector, each setup step, and the unavailable on-device choice are lockable with unit tests and previews)
- Agent UI: none

### web

- deferred

## History

- 2026-10-13: proposed
- 2026-10-13: written as the finished setup — selector, then Flim account and plan (promo or subscription), custom server, on-device when the device can. No interim wiring in this decision
- 2026-10-13: failure statuses live in decision 004; this decision does not restate them
- 2026-10-13: custom-server listener is decision 006
- 2026-10-13: on-device listener, and the test behind "the device can", are decision 007
- 2026-10-13: Android implements the same three choices as iOS (Flim, custom server, on this device)
