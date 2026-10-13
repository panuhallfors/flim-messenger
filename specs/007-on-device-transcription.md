---
id: 007
slug: on-device-transcription
title: On-device transcription
status: proposed
created: 2026-10-13
amends: [004]
platforms: [ios, android]
---

# 007 — On-device transcription

## Intent

When the user has finished on-device setup (decision 005) and that method is in use, Flim produces the title, summary, and transcript on the phone itself. The voice message is not sent to a server for this method. The phone can take the method only when it can both transcribe speech and write a title and summary locally. That path no longer uses decision 004's mock.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] With on-device as the method in use, a voice message's title, summary, and transcript are produced on that phone. The audio is not sent to Flim, to a custom server, or to any other service for this method.
- [ ] The same brief presentation, race, room notice, and failure statuses from decision 004 still apply; only the source of the words changes on this path.
- [ ] On this device can be turned on only when the phone can both transcribe speech and write a title and summary without leaving the device, and those system models are ready. Otherwise the choice stays visible and says why it cannot be turned on.
- [ ] A result that is not a transcript plus a title and a summary is not shown, and nothing is sent to the room.
- [ ] Flim and custom server are unchanged by this decision.

## Non-goals

- The settings selector, the quality note, and the Flim or custom-server setups (decision 005). Custom server's listener is decision 006.
- Shipping a speech or language model inside the app.
- Sending the audio, the transcript, or the room context off the device to fill a gap, including a vendor private cloud or a hybrid local/cloud model.
- The hosted Flim transcription service.
- Changing decision 004's bubble, wait, race, companion notice, or failure status list.
- Transcribing audio that is not a voice message.
- A promise that the operating system will let a long on-device job finish after the app is suspended (decision 004 already covers that limit).

## Notes

Decision 005 chooses the method. This decision is the listener for on-device, and the test behind that screen's "the device can".

The product design is the same on iOS and Android. Platform notes name today's system models only so the gate stays realistic; a later OS that widens the set does not need a new decision while the product rule holds. The transcription section on both platforms is decision 005's full selector: Flim, custom server, and on this device.

Capable phones are a minority. Speech on its own is common. A local model that can title and summarise a transcript is not: it is the current on-device system model, and it is missing on many phones that can still transcribe. The choice stays visible and explains the gap.

New user-facing strings belong to the Flim catalog (decision 002).

Design (replay-canonical with this decision): [`007-on-device-transcription.design.md`](007-on-device-transcription.design.md)

## Verification

### ios

- Tests: none yet (availability reasons, and mapping a transcript plus a title and summary or a failure onto decision 004, are lockable with unit tests against a fake on-device engine)
- Agent UI: none

### android

- Tests: none yet (same cases as iOS, against a fake on-device engine)
- Agent UI: none

### web

- deferred

## History

- 2026-10-13: proposed
- 2026-10-13: on-device title and summary use the same prompt wording as decision 006
- 2026-10-13: Android uses decision 005's full selector, not an on-device-only section
