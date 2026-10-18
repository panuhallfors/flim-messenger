---
id: 009
slug: transcription-setup-prompts
title: Transcription setup prompts
status:
  ios: proposed
  android: proposed
created: 2026-10-14
amends: [004, 005]
platforms: [ios, android]
---

# 009 — Transcription setup prompts

## Intent

The user is asked to finish choosing a voice-message transcription method during post-login onboarding, and sees a stronger call-to-action the first time they meet a voice message while none is set up. They can skip the onboarding ask. Whenever transcription is not set up, the voice-message error action opens setup — it does not offer try again.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] After the existing post-login onboarding steps, the user is asked to set up voice-message transcription before they reach the main app, unless a method is already in use.
- [ ] That ask can be skipped. Skipping does not set up a method.
- [ ] The first time the user sees a voice message (sent or received) while no transcription method is in use, that voice message shows a clear setup call-to-action.
- [ ] Whenever transcription is not set up, the voice message’s error action opens method setup — not try again.
- [ ] Opening that way, or finishing setup from the onboarding ask, lands the user in decision 005’s method choice and its setup. Afterwards a method in use behaves as 004 and 005 already specify.

## Non-goals

- Changing which methods exist, how each method’s setup works, or where the lasting settings entry lives (decision 005).
- Changing decision 004’s race, wait, room notice, other failure statuses, or who may publish — only how **not set up** is acted on, plus the one-shot stronger CTA.
- Replacing analytics, notification, identity, or app-lock onboarding.
- Forcing a method before the user can use the rest of the app.
- Showing the stronger first-voice-message call-to-action again after it has been shown once (the quiet not-set-up status and setup link remain until a method is in use).

## Notes

Upstream Element X has no transcription method. Post-login onboarding already runs identity confirmation, optional app lock, analytics, and notification permissions; this decision adds one more step at the end of that sequence.

New user-facing strings belong to the Flim catalog (decision 002).

Design (replay-canonical with this decision): [`009-transcription-setup-prompts.design.md`](009-transcription-setup-prompts.design.md)

## Verification

### ios

- Tests: none yet (onboarding step order, skip, first voice-message CTA, and not-set-up → setup are lockable with unit tests and previews)
- Agent UI: none

### android

- Tests: none yet (same outcomes as iOS)
- Agent UI: none

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-14: proposed
- 2026-10-14: not-set-up always uses a setup link (amends 004); stronger CTA only on the first voice message
- 2026-10-14: platforms = ios, android
