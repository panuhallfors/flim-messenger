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

Until a result is adopted, the block shows a quiet in-progress state (waiting on the sender, or producing a result on device). Those two waits look the same.

When an attempt fails and nothing has been adopted yet, the same place where the title, summary, and transcript would appear shows an **error** instead — not a blank, and not a separate timeline row. The player stays.

**Keep it simple.** No codes, no stack traces, no “error 429” style chrome. One quiet block under the player, same visual weight as the brief would have had:

1. **Status** — one short plain-language line from the Flim catalog (what happened).
2. **Detail** (optional) — at most one more line. If the service returned a short human-readable note, show the **start** of that note and truncate the rest with an ellipsis so it stays a single line. Drop the detail entirely when it is empty, technical, or a duplicate of the status line. Never invent jargon to fill the slot.
3. **Action** — its own **tappable** control (text-button / link style), labelled from the catalog. Not buried inside the status sentence, and not a second screen. For most statuses the label is “Try again” and the tap starts a new attempt on this device. For **not set up**, the label opens setup instead (below) — another attempt cannot succeed until a method is in use.

Trying again is allowed only when nothing has been adopted yet; a result that is already showing stays.

Every failure uses that same layout. The status is one of these:

1. **Generic failure** — the transcript could not be made (English source: “Couldn't make a transcript.”).
2. **Not set up** — transcription is not set up (English source: “Transcription isn't set up.”). The action is a setup link (not Try again): it opens decision 005’s transcription selector / setup. Decision 009 adds a stronger one-shot call-to-action the first time this status appears on a voice message; afterwards this quiet status + setup link remains.
3. **No reply** — the service could not be reached (English source: “Couldn't reach the transcription service.”). Silence, a timeout, and a connection that drops are this status.
4. **Service fault** — the service replied and failed (English source: “The transcription service failed.”).
5. **Allowance used up** — the allowance is used up. For Flim, say that a larger transcription-service subscription is needed. For a server the user named, say that this key's allowance is used up. No store or plan-change flow on the bubble — only that message. Try again may succeed after the plan changes.
6. **Too many requests** — try again shortly (English source: “Too many requests. Try again shortly.”). This is not the allowance message, even when the host uses one refusal for both. A refusal that cannot be read as an empty allowance stays this status, so a throttle never asks the user to buy a larger plan.
7. **Sign-in rejected** — the key or Flim sign-in was refused (English source: “Sign-in was refused.”). Fix it in Settings.
8. **Setup is wrong** — the server or the model settings look wrong (English source: “The transcription settings look wrong.”).
9. **Unusable reply** — the service answered in a shape that is not a title, a summary, and a transcript (English source: “The transcription service replied in an unexpected way.”).

Tell “no reply” apart from “the service replied and failed.” Transcription and summary are one attempt. If the transcript step succeeds and the summary step fails, nothing is adopted and no notice is sent. Try again runs both steps again.

A failure never publishes a companion notice. Other people in the room do not see the error. On a Flim-marked send that fails on the sending device, recipients still wait 15 seconds, then produce a result on their own device if no trusted brief arrived.

The conversation-list preview for that voice message uses the title once a result is adopted. Before that, and after a failure with nothing adopted, it stays the upstream voice-message preview.

### Where the words come from

Every platform calls an in-app service. Input is the voice message's audio plus **room context** (below). Output is a title, a summary, and a transcript.

- Title is at most 50 characters. A longer title is truncated on a character boundary before it is shown or sent.
- Summary is about two sentences, and is not a copy of the transcript.
- Transcript is the full text.

#### Room context (every call)

The service needs enough surrounding conversation to place what was said. Every call includes, built by the caller before invoking the service:

1. **Reply target** — if this voice message is a reply, that parent message as one context item (and a flag that it is the reply target).
2. **Recent messages** — the five most recent messages **before** this voice message, in the same timeline the voice message sits in (the room's main timeline, or the thread if the voice message is in a thread). Oldest of the five first, then toward the voice message. Fewer than five is fine when the timeline is shorter.

If the reply target is already among those five, it still appears once as content; the reply-target flag is what marks the relation. This voice message itself is not included in the five.

Each context item is **text only**. Never send prior messages' media bytes (audio, image, video, file) to the service. Per item:

| Kind | What is sent |
|------|----------------|
| Ordinary text message | The message body text |
| Voice message that already has an adopted transcript (local or trusted) | That transcript text (and title/summary if adopted — enough that the service sees the spoken words as text) |
| Voice message without an adopted transcript | A short kind label only, e.g. that it is a voice message — not the audio |
| Image (and other non-text media: video, file, sticker, location, etc.) | A short kind label only, e.g. that it is an image — not the media. If the event has a text caption, that caption text may be included; the media itself is not |
| Redacted / unavailable | A short kind label that it is unavailable — not an attempt to recover content |

Sender display name (or a stable sender id) may accompany each item so the service can tell speakers apart. That is still text metadata, not media.

Gathering this context is the caller's job and stays outside the mock/engine swap. Both implementations receive the same context shape. The mock may ignore the context when choosing sample text or a failure; the real engine may use it. Callers always pass it.

#### Contract (replaceable implementation)

Every caller talks to that service and to nothing behind it. The bubble, the 15-second wait, the on-device race, retry, and the companion notice do not know whether the service is the mock. There is no second path for "sample" versus "real".

The contract is fixed now, so a later decision can replace the implementation without editing those callers:

- Start from the voice message's audio **and** the room context above.
- Finish with a title, a summary, and a transcript.
- Fail as one of the statuses above, so the bubble can show the matching error and try again.
- Be cancelled, so an in-flight attempt can be dropped when an earlier result wins.

The call is asynchronous from the start. The mock may return immediately; a later engine may take time. Callers already wait either way.

Title truncation to 50 characters sits outside the service, in the one place that shows and sends the result. Both implementations share it. Mapping a service failure to one of the statuses above also sits outside the engine, in that same place.

The service in this decision is a mock. It does not read the audio or the context. On a successful call it returns a fixed title, summary, and transcript. The fixed text says that it is sample text, not what was spoken. **Every third call fails** with a generic failure, so the shipping app exercises the error UI in real rooms without a real engine. The mock can also return any of the other statuses (for previews and tests) under the same failure contract. The shipping app calls this mock in real rooms, not only in tests or previews: on success the sender's bubble shows the sample and the companion notice carries the sample title and summary; on failure the sender's bubble shows the error and no notice is sent.

A later decision replaces the mock by swapping which implementation is constructed. It does not edit the bubble, the race, the failure UI, context gathering, or the notice. Credentials, a model, quota accounting, or a network client that the engine needs live inside that implementation. A new user-visible behaviour is a new decision, not a change to these callers.

The race below is per device. Only the device that sent the voice message publishes a brief to the other people in the room. If a recipient produces a result, it stays on that device.

### Sending

Flim sends the voice message immediately. It does not wait for the service.

Every voice message Flim sends carries a marker that a brief from this sender may follow. The marker is set even when the brief is not ready, and even if the brief later fails. Receivers use it only to decide whether to wait.

The sending device starts the service immediately. When the result returns, that device adopts it if it has not already adopted one, and publishes one companion notice if the room does not already have a trusted brief for this voice message. Publishing retries the same adopted result; it does not call the service again. That notice is how the original sender's title and summary reach the other people in the room. When the service fails, nothing is adopted, no notice is published, and the bubble shows the matching error.

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

### On app launch (after a close or kill)

Incomplete work is remembered per voice message on this device. When the app starts again (or returns to a session that can run the service), unfinished briefs are handled as follows.

**Received** (this device did not send the voice message — including another device of the same account):

- If nothing is adopted yet and a brief is still owed (no trusted brief in the room, or the wait already ended without one): continue — start or resume the service. No 15-second wait again if that wait already completed before the close; if the wait was still running, finish the remaining wait, then continue as on receive.
- A received result stays on this device. Relaunch never publishes for a received message.

**Sent** (this device sent the voice message):

- Resume only while the voice message is **younger than 10 minutes** (from its send time).
  - No adopted result and no trusted brief in the room → start the service immediately (no 15-second wait).
  - Adopted result not yet published → publish that result; do not call the service again.
- **10 minutes or older:** do not auto-start or resume the service, and do not publish a late companion notice. Recipients should not get a Flim title/summary in the room long after they already heard the audio. The bubble may show the quiet failure (or stay without a brief); the user can still tap Try again for a **local** result on this device only — that retry must not publish once the send is past 10 minutes.

The 10-minute limit applies only to **outgoing** auto-resume and to **publishing**. It does not limit received local briefs.

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

Producing the brief and publishing the notice are part of handling the voice message, not part of keeping the room open.

Upstream already hands outgoing messages to a send queue, and when the app enters the background it starts a limited background interval before pausing client services. This decision requires the same kind of attempt for an in-flight brief: keep producing the result (and, on the sending device, publish the notice while still within the 10-minute window), across leaving the conversation and across the app going to the background.

The scheduler is not chosen here (extending that existing interval, a later refresh task, or something else). Closing or backgrounding the app must not be treated as a reason to drop an in-flight attempt early. If the system ends the process before work finishes, the next launch follows **On app launch** above: received work resumes; outgoing work resumes only under 10 minutes from send. If an outgoing notice never published in time, there is no late notice — recipients use their 15-second wait and their own on-device result.

## Platform notes

- iOS: the Rust SDK's typed timeline content drops unknown content keys. Reading `com.hallfors.panu.flim.voice` and `com.hallfors.panu.flim.voice_summary` needs the original event JSON (already exposed on timeline debug info) or a small SDK pass-through. The typed voice-message model is not enough.
- iOS: 50 characters means extended grapheme clusters, so an emoji counts as one.
- iOS: status, attribution, each failure status above, and “Try again” strings are Flim catalog keys (decision 002), not `Localizable.strings` / `Untranslated.strings`. English source for Flim's allowance status must state that a larger transcription-service subscription is needed.
- iOS background: the existing pieces to hang the attempt on are the SDK send queue and `AppCoordinator`'s `beginBackgroundTask` when entering background. Which of those, or a new task, actually runs the service is an implementation choice.
- iOS: one protocol for the service, including room context on every call and a typed failure for each status above. This decision adds the mock as its implementation (success with sample text, every third call a generic failure, the other statuses available for previews/tests). The composition root is the only place that names the mock. A later engine conforms to the same protocol and is selected there.
- iOS: context items are built from timeline / local adopted briefs before the service call; do not download prior media solely to feed the service.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: only the sending device publishes to the room; a recipient's result stays on device. Wait for the sender's brief is 15 seconds. Mock service described as fixed sample text, not as a claim about the audio
- 2026-10-13: the mock runs in real rooms; sample title and summary are sent to other participants until a later decision replaces the service
- 2026-10-13: one service contract (start, finish, fail, cancel). Replacing the mock does not change callers, the bubble, the race, or the notice
- 2026-10-13: failures show in the brief slot with try again; no notice on failure; quota exhausted is a distinct message; mock fails every third call (generic) and can return quota for previews/tests
- 2026-10-13: every service call includes reply target + five preceding messages as text-only context; prior media sent only as kind labels (plus caption text if any); existing transcripts sent as text
- 2026-10-13: failure UI simplified — catalog status, optional one-line truncated human detail, separate tappable Try again; no tech codes
- 2026-10-13: relaunch resumes unfinished received briefs; outgoing resume/publish only within 10 minutes of send (no late room notice); Try again after that stays local-only
- 2026-10-13: amended by 005 — the user chooses the method in settings and finishes its setup before a brief is attempted
- 2026-10-13: failure statuses cover not set up, no reply, service fault, allowance, too many requests, sign-in rejected, wrong setup, and an unusable reply, besides the generic failure
- 2026-10-13: amended by 006 — custom-server method uses an OpenAI-compatible listener instead of the mock; other methods unchanged
- 2026-10-13: amended by 007 — on-device method uses the on-device listener instead of the mock; other methods unchanged
- 2026-10-14: auth failure status renamed to Sign-in rejected (covers API key and Flim session); English source “Sign-in was refused.”
- 2026-10-14: amended by 009 — not-set-up action is a setup link into decision 005, not Try again; first encounter may use 009’s stronger one-shot CTA
