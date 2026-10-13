---
decision: 004
slug: voice-message-brief
created: 2026-10-13
---

# 004 — Voice message title, summary, and transcript (design)

Companion to `004-voice-message-brief.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Upstream sends a voice message as an audio message and shows a player. It does not title, summarise, or transcribe it. Element X iOS does not draw a caption on the voice-message bubble, so putting the brief only in the audio event's caption would leave other Element X clients with silence.

This decision adds the brief around that player, and a companion room event so other Matrix clients still see the title and the summary. The words are produced by an in-app service. The service shipped here is a mock that returns fixed sample text; a later decision replaces it with one that reads the audio. The bubble, the race, and the room event stay as specified here.

## Product design

### What Flim shows

The voice-message block keeps the existing player. With it, in this order:

1. **Title** — one line, at most 50 characters.
2. **Summary** — a couple of sentences about what was said.
3. **Transcript** — the full text.

All three are part of that voice message. Flim does not also show them as their own timeline row.

Until a result is adopted, the block shows a quiet in-progress state (waiting on the sender, or producing a result on device). Those two waits look the same. A failed attempt that adopted nothing shows a quiet failure and a way to try again on this device. Trying again is a new attempt only when nothing has been adopted yet; a result that is already showing stays.

The conversation-list preview for that voice message uses the title once a result is adopted. Before that, it stays the upstream voice-message preview.

### Where the words come from

Every platform calls an in-app service. Input is the voice message. Output is a title, a summary, and a transcript.

- Title is at most 50 characters. A longer title is truncated on a character boundary before it is shown or sent.
- Summary is about two sentences, and is not a copy of the transcript.
- Transcript is the full text.

Every caller talks to that service and to nothing behind it. The bubble, the 15-second wait, the on-device race, retry, and the companion notice do not know whether the service is the mock. There is no second path for "sample" versus "real".

The contract is fixed now, so a later decision can replace the implementation without editing those callers:

- Start from the voice message's audio.
- Finish with a title, a summary, and a transcript.
- Fail, so the bubble can show the quiet failure and try again.
- Be cancelled, so an in-flight attempt can be dropped when an earlier result wins.

The call is asynchronous from the start. The mock may return immediately; a later engine may take time. Callers already wait either way.

Title truncation to 50 characters sits outside the service, in the one place that shows and sends the result. Both implementations share it.

The service in this decision is a mock. It does not read the audio. It returns a fixed title, summary, and transcript. The fixed text says that it is sample text, not what was spoken. The shipping app calls this mock in real rooms, not only in tests or previews: the sender's bubble shows the sample, and the companion notice carries the sample title and summary to the other people in the room.

A later decision replaces the mock by swapping which implementation is constructed. It does not edit the bubble, the race, or the notice. Credentials, a model, or a network client that the engine needs live inside that implementation. A new user-visible behaviour is a new decision, not a change to these callers.

The race below is per device. Only the device that sent the voice message publishes a brief to the other people in the room. If a recipient produces a result, it stays on that device.

### Sending

Flim sends the voice message immediately. It does not wait for the service.

Every voice message Flim sends carries a marker that a brief from this sender may follow. The marker is set even when the brief is not ready, and even if the brief later fails. Receivers use it only to decide whether to wait.

The sending device starts the service immediately. When the result returns, that device adopts it if it has not already adopted one, and publishes one companion notice if the room does not already have a trusted brief for this voice message. Publishing retries the same adopted result; it does not call the service again. That notice is how the original sender's title and summary reach the other people in the room.

Another signed-in device of the same account did not send the message. It follows the receive path and does not publish.

### Receiving

When a voice message arrives:

- **No Flim marker.** Start the service immediately.
- **Flim marker.** Wait 15 seconds for a trusted brief from the voice message's sender.
  - If that brief arrives during the wait, adopt it and do not start the service.
  - If the wait ends with nothing adopted, start the service.

The first result to become ready on this device is adopted. That includes a trusted brief that arrives while the local service is still running: the brief is ready first, the local result is discarded when it later finishes. A trusted brief that arrives after a local result was already adopted is discarded. Its text is not shown. Flim still does not draw that notice as its own row.

A recipient does not publish. The result they produce is for themselves on that device, and the other people in the room do not receive it.

**Trusted** means: the notice is the companion defined below, its relation points at this voice message, and its sender is the same user as the voice message's sender. Anything else with the marker stays an ordinary message — it is not attached to the player, and it is not hidden. The marker must not be a way for someone else in the room to overwrite a transcript or to hide a message.

An adopted result is kept on device for that voice message, so a later launch does not start another attempt and does not swap in a late notice.

The sending device, on a later launch, if it still has no adopted result and the room has no trusted brief: start the service immediately (no 15-second wait). If it already adopted a result that was never published, publish that result and do not call the service again.

### What other clients see

They see a normal voice message, then a notice that replies to it. The notice's visible text is only:

1. the title
2. a blank line
3. the summary
4. a blank line
5. a short attribution that names Flim

The attribution is the sender's Flim-catalog string for "made with Flim" (decision 002). English source: "Made with Flim". The full transcript is not in the visible text.

The notice is an `m.notice`, so default push rules do not raise a second notification for it. The voice message still notifies as it does today. Clients that have turned notice suppression off may notify. The notice is its own timeline event, so clients that count every event will count it.

Flim does not draw a trusted companion notice as its own timeline row, whether or not its text was the result Flim adopted.

If the voice message is in a thread, the notice is in that same thread.

When the sender redacts the voice message, Flim also redacts the companion notice they published for it, and drops any result stored on device for that message. The transcript should not remain in the room after the audio is removed.

### Protocol

Namespace is the product id from decision 000a: `com.hallfors.panu.flim`. Custom keys live in the event `content`, which is encrypted in encrypted rooms. They are not placed in `unsigned`.

**Voice message** (Flim-sent), in addition to the ordinary voice-message content:

```json
"com.hallfors.panu.flim.voice": { "v": 1 }
```

Presence is the whole signal. No brief is stored on the audio event.

**Companion notice:**

```json
{
  "msgtype": "m.notice",
  "body": "<title>\n\n<summary>\n\nMade with Flim",
  "m.relates_to": {
    "rel_type": "com.hallfors.panu.flim.voice_summary",
    "event_id": "$voiceEventId",
    "m.in_reply_to": { "event_id": "$voiceEventId" }
  },
  "com.hallfors.panu.flim.voice_summary": {
    "v": 1,
    "title": "<title>",
    "summary": "<summary>",
    "transcript": "<transcript>"
  }
}
```

`body` is plain text. There is no `formatted_body`. The transcript exists only under `com.hallfors.panu.flim.voice_summary`. The visible attribution in `body` is the sender's localized catalog string; the English source above is the fallback shape.

One notice per voice message. A later correction is not sent: a result that finishes later is discarded, not edited in.

### After the app leaves the foreground

Producing the brief and publishing the notice are part of sending a voice message, not part of keeping the room open.

Upstream already hands outgoing messages to a send queue, and when the app enters the background it starts a limited background interval before pausing client services. This decision requires the same kind of attempt for an in-flight brief on the sending device: keep producing the result, then publish the notice, across leaving the conversation and across the app going to the background.

The scheduler is not chosen here (extending that existing interval, a later refresh task, or something else). What is fixed is the product rule: closing or backgrounding the app must not be treated as a reason to drop the attempt. If the system ends the process before the notice is published, there is no notice. Recipients then hit the 15-second wait and produce their own result on device. The next launch of the sending device follows the resume rule above.

## Platform notes

- iOS: the Rust SDK's typed timeline content drops unknown content keys. Reading `com.hallfors.panu.flim.voice` and `com.hallfors.panu.flim.voice_summary` needs the original event JSON (already exposed on timeline debug info) or a small SDK pass-through. The typed voice-message model is not enough.
- iOS: 50 characters means extended grapheme clusters, so an emoji counts as one.
- iOS: status and attribution strings are Flim catalog keys (decision 002), not `Localizable.strings` / `Untranslated.strings`.
- iOS background: the existing pieces to hang the attempt on are the SDK send queue and `AppCoordinator`'s `beginBackgroundTask` when entering background. Which of those, or a new task, actually runs the service is an implementation choice.
- iOS: one protocol for the service. This decision adds the mock as its implementation. The composition root is the only place that names the mock. A later engine conforms to the same protocol and is selected there.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: only the sending device publishes to the room; a recipient's result stays on device. Wait for the sender's brief is 15 seconds. Mock service described as fixed sample text, not as a claim about the audio
- 2026-10-13: the mock runs in real rooms; sample title and summary are sent to other participants until a later decision replaces the service
- 2026-10-13: one service contract (start, finish, fail, cancel). Replacing the mock does not change callers, the bubble, the race, or the notice
