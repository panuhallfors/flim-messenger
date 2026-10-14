---
id: 004
slug: voice-message-brief
title: Voice message title, summary, and transcript
status: proposed
created: 2026-10-13
amends: []
platforms: [ios, android]
---

# 004 — Voice message title, summary, and transcript

## Intent

A voice message in Flim shows a one-line title and a short summary with the audio by default; the full transcript is on that same message behind a toggle. Other Matrix clients see the title and the summary in connection with that voice message, not the full transcript. The words come from an in-app service. This decision is the presentation and how a result travels in the room. A mock fills that service only until real listener decisions replace it on each path (006, 007, 008).

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] Once a result is ready, Flim shows a one-line title and a short summary with that voice message by default; the full transcript is available on that same message (not as a separate timeline row), and the visible brief text can be selected and copied.
- [ ] The voice message still plays as a voice message, whether or not that text is ready.
- [ ] A newly arriving voice message sent from Flim is one a recipient can expect a sender's result for: Flim waits briefly for it, and only then produces a result on device.
- [ ] A newly arriving voice message that did not come from Flim starts producing a title, summary, and transcript on the recipient's device straight away.
- [ ] Opening a room or scrolling older voice messages into view does not start producing a title, summary, or transcript for those messages.
- [ ] The first result that becomes ready for a voice message is the one Flim keeps. A result that finishes later does not replace it.
- [ ] Only the person who sent the voice message shares a title and summary with the rest of the room. A title, summary, or transcript that someone else produces stays on their own Flim.
- [ ] In another Matrix client, the title and the summary show up in connection with the voice message; that visible text starts with a short note that Flim Messenger produced the summary, and the full transcript is not visible message text. If the voice message was in a thread, that notice is in the same thread.
- [ ] A long transcript still gets a companion notice into the room: the notice’s stored transcript may end with an ellipsis so the event fits; the sending device keeps the full transcript on that voice message. Size alone does not leave the room without a notice or show a failure for the brief.
- [ ] A trusted companion notice never marks the room unread on Flim by itself — there is nothing visible in the timeline to read for that event.
- [ ] Sending a voice message and immediately leaving the app does not cancel the attempt to produce that result and to share the title and summary with the room while the attempt is still timely.
- [ ] After the app is closed mid-work, opening it again continues unfinished briefs for voice messages this device received. For a voice message this device sent, unfinished work resumes only while that send is still recent — so a title and summary do not appear in the room long after people already heard the audio.
- [ ] While no real listener yet exists for the method in use, a successful result in a real room may be the mock’s fixed sample text (and other people may see that sample title and summary). Once that method’s real listener is in the replay (006, 007, or 008), that path never uses the mock — real rooms get a real result, a real failure, or not-set-up, not sample text.
- [ ] When producing the title, summary, and transcript fails and nothing has been adopted yet, Flim shows a plain-language error in that place on the voice message, offers a tappable next step (try again, or set up when transcription is not configured), does not send a brief to the room, and leaves the voice message playable.
- [ ] When that failure is that the transcription quota is used up, the error says clearly that a larger transcription-service subscription is needed.

## Non-goals

- The speech-to-text or summarisation engine: model, on-device versus network, languages, quality, pricing, or how a larger subscription is bought. A later decision replaces the mock implementation only. Call sites, the bubble, the wait, the failure UI (including the quota message), and the room event stay as they are. This decision's text is not a transcript of the spoken audio.
- Anyone but the voice message's sender publishing a brief into the room. A recipient's result stays on their own device.
- Showing the full transcript as visible message text to other clients.
- Hand-editing the title, summary, or transcript.
- Replacing a result that is already shown, including when the sender's result arrives after a local one.
- Holding the voice message back until the text is ready.
- Transcribing audio or video files that are not voice messages.
- Automatically starting a brief for a voice message that was already in the room's history when this device processes it (pagination, opening an old room, or any other path that only brings past events onto the screen). Catch-up of messages that arrived while the app was away still counts as newly arriving.
- Sending prior messages' media (audio, image, video, file bytes) into the transcription service. Context for the service is text only — message text, an existing transcript, or a short kind label for media.
- A guarantee that the brief finishes after the operating system suspends or kills the app. The app attempts to continue; the system may still stop it.

## Notes

Upstream voice messages are ordinary audio messages with Element's voice-message marker. Element X iOS draws the player only — a caption on that message is not shown on the bubble — so a brief other clients can actually read has to be its own event. Upstream already has a send queue and a limited background interval when the app leaves the foreground; this decision extends that *attempt* to the brief, without choosing the scheduler.

New user-facing strings belong to the Flim catalog (decision 002), not Element's `Localizable` files.

Decision 005 adds the settings choice of method. A brief is attempted once a method is set up. The failure statuses are specified here.

Design (replay-canonical with this decision): [`004-voice-message-brief.design.md`](004-voice-message-brief.design.md)

## Verification

### ios

- Tests: none yet (the race, the marker, notice folding, and bubble states are lockable with unit tests and previews against the mock service)
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-13: proposed
- 2026-10-13: sender-only sharing called out in acceptance; recipient results stay on device. Wait for a sender's brief is 15 seconds (design)
- 2026-10-13: the mock's fixed sample text is what real rooms show and send until a later decision replaces the service
- 2026-10-13: the later engine replaces the mock implementation only; callers stay put
- 2026-10-13: service failures show locally where the brief would be; no room notice on failure; quota-exhausted is a distinct, clear message
- 2026-10-13: service call includes reply target + last five messages as text-only context (design); prior media never sent as media
- 2026-10-13: failure UI — plain language, tappable try again (design)
- 2026-10-13: on relaunch, resume unfinished received briefs; resume outgoing only within 10 minutes of send (design)
- 2026-10-13: amended by 005 — the user chooses the transcription method and finishes its setup before a brief is attempted
- 2026-10-13: failure statuses in the design cover each plain-language case, not only a generic failure and a used-up allowance
- 2026-10-13: amended by 006 — when custom server is the method in use, that OpenAI-compatible listener replaces the mock on that path only
- 2026-10-13: amended by 007 — when on-device is the method in use, the on-device listener replaces the mock on that path only
- 2026-10-14: amended by 009 — when transcription is not set up, the bubble action opens setup instead of try again
- 2026-10-14: recipient auto-brief only for newly arriving voice messages; scrolling or opening history does not start the service
- 2026-10-14: companion notice body for other clients starts with “Summary by Flim Messenger” (replaces trailing “Made with Flim”)
- 2026-10-14: platforms = ios, android
- 2026-10-14: companion notice in a thread uses m.thread (with in_reply_to the voice event); custom voice_summary relation on the main timeline only
- 2026-10-14: Not set up only when no method is in use (on-device not-ready is generic failure per 007)
- 2026-10-14: mock only until each path’s real listener (006 / 007 / 008); no separate amending id
- 2026-10-14: title/summary limits aligned with 006/007 prompt caps (design)
- 2026-10-14: companion notice truncates an oversized transcript with an ellipsis so the event fits under the Matrix size limit; sender keeps the full transcript locally (design)
- 2026-10-14: trusted companion never counts toward Flim’s client-side unread state (design)
- 2026-10-14: adopted brief shows title + summary by default; tap toggles summary ↔ full transcript; brief text is selectable and copyable; disclosure hint for expand/collapse (design)
