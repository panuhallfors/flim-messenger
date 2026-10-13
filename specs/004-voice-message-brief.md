---
id: 004
slug: voice-message-brief
title: Voice message title, summary, and transcript
status: proposed
created: 2026-10-13
amends: []
platforms: [ios]
---

# 004 — Voice message title, summary, and transcript

## Intent

A voice message in Flim shows a one-line title, a short summary, and a full transcript with the audio. Other Matrix clients see the title and the summary in connection with that voice message, not the full transcript. The words come from an in-app service. This decision is the presentation and how a result travels in the room. The service that actually listens to the audio is a later decision; until then the app uses a mock that returns fixed sample text.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] Once a result is ready, Flim shows a one-line title, a short summary, and the full transcript with that voice message — not as a separate message of their own.
- [ ] The voice message still plays as a voice message, whether or not that text is ready.
- [ ] A voice message sent from Flim is one a recipient can expect a sender's result for: Flim waits briefly for it, and only then produces a result on device.
- [ ] A voice message that did not come from Flim starts producing a title, summary, and transcript on the recipient's device straight away.
- [ ] The first result that becomes ready for a voice message is the one Flim keeps. A result that finishes later does not replace it.
- [ ] Only the person who sent the voice message shares a title and summary with the rest of the room. A title, summary, or transcript that someone else produces stays on their own Flim.
- [ ] In another Matrix client, the title and the summary show up in connection with the voice message, with a short note that Flim produced them, and the full transcript is not visible message text.
- [ ] Sending a voice message and immediately leaving the app does not cancel the attempt to produce that result and to share the title and summary with the room.
- [ ] Until a later decision supplies a service that listens to the audio, the title, summary, and transcript in real rooms are the fixed sample text. Other people in the room see that sample title and summary.

## Non-goals

- The speech-to-text or summarisation engine: model, on-device versus network, languages, or quality. A later decision replaces the mock implementation only. Call sites, the bubble, the wait, and the room event stay as they are. This decision's text is not a transcript of the spoken audio.
- Anyone but the voice message's sender publishing a brief into the room. A recipient's result stays on their own device.
- Showing the full transcript as visible message text to other clients.
- Hand-editing the title, summary, or transcript.
- Replacing a result that is already shown, including when the sender's result arrives after a local one.
- Holding the voice message back until the text is ready.
- Transcribing audio or video files that are not voice messages.
- A guarantee that the brief finishes after the operating system suspends or kills the app. The app attempts to continue; the system may still stop it.

## Notes

Upstream voice messages are ordinary audio messages with Element's voice-message marker. Element X iOS draws the player only — a caption on that message is not shown on the bubble — so a brief other clients can actually read has to be its own event. Upstream already has a send queue and a limited background interval when the app leaves the foreground; this decision extends that *attempt* to the brief, without choosing the scheduler.

New user-facing strings belong to the Flim catalog (decision 002), not Element's `Localizable` files.

Design (replay-canonical with this decision): [`004-voice-message-brief.design.md`](004-voice-message-brief.design.md)

## Verification

### ios

- Tests: none yet (the race, the marker, notice folding, and bubble states are lockable with unit tests and previews against the mock service)
- Agent UI: none

### android

- deferred

### web

- deferred

## History

- 2026-10-13: proposed
- 2026-10-13: sender-only sharing called out in acceptance; recipient results stay on device. Wait for a sender's brief is 15 seconds (design)
- 2026-10-13: the mock's fixed sample text is what real rooms show and send until a later decision replaces the service
- 2026-10-13: the later engine replaces the mock implementation only; callers stay put
