---
decision: 004
slug: voice-message-brief
created: 2026-10-13
---

# 004 — Voice message title, summary, and transcript (design)

Companion to `004-voice-message-brief.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Upstream sends a voice message as an audio message and shows a player. It does not title, summarise, or transcribe it. Element X iOS does not draw a caption on the voice-message bubble, so putting the brief only in the audio event's caption would leave other Element X clients with silence.

This decision adds the brief around that player, and a companion room event so other Matrix clients still see the title and the summary. The words are produced by an in-app service. This decision defines the service contract and ships a **mock** as the interim implementation. Decisions 006, 007, and 008 each replace the mock on their path when they are in the replay. The bubble, the race, and the room event stay as specified here.

## Product design

### What Flim shows

The voice-message block keeps the existing player. With an adopted result, Flim draws:

1. **Title** — one line, at most four words and at most 35 characters. Always visible once a result is adopted.
2. **Body** — either the **summary** or the **transcript**, never both at once:
   - **Summary** (default) — short: at most four short sentences, and at most 20 words, and at most 160 characters; not a copy of the transcript.
   - **Transcript** — the full text.

**Default.** After a result is adopted, show title + summary. Do not put the full transcript in the timeline until the user asks for it.

**Toggle.** A tap on the brief text (title or body) or on the disclosure control switches the body between summary and transcript. Tap again to switch back. The title stays. There is no state that shows summary and transcript together. The toggle is local UI state for that bubble; it does not change what is stored or published.

**Visual hint.** Always show a small chevron next to the brief so expand/collapse is discoverable without relying on a guess that the text is tappable. No text label (“Show transcript” / “Hide transcript” or similar) — chevron only.

- **Summary showing** — chevron pointing down (expand to transcript).
- **Transcript showing** — same control, chevron pointing up (collapse to summary).

The chevron stays visually quiet (same weight as secondary timeline chrome, not a primary button). It must remain visible in both states so collapse is as obvious as expand. Tapping the chevron triggers the same toggle as tapping the brief text.

**Select and copy.** Title, summary, and transcript (whichever is showing) are ordinary selectable text. The user can select a range and copy it with the platform’s usual copy action. Long-press / drag-to-select takes priority over the toggle so selecting text does not flip the body.

All of this stays on that voice message. Flim does not also show title, summary, or transcript as their own timeline row.

Until a result is adopted, the block shows a quiet in-progress state (waiting on the sender, or producing a result on device). Those two waits look the same.

When an attempt fails and nothing has been adopted yet, the same place where the title and body would appear shows an **error** instead — not a blank, and not a separate timeline row. The player stays.

**Keep it simple.** No codes, no stack traces, no “error 429” style chrome. One quiet block under the player, same visual weight as the brief would have had:

1. **Status** — one short plain-language line from the Flim catalog (what happened).
2. **Detail** (optional) — at most one more line. If the service returned a short human-readable note, show the **start** of that note and truncate the rest with an ellipsis so it stays a single line. Drop the detail entirely when it is empty, technical, or a duplicate of the status line. Never invent jargon to fill the slot.
3. **Action** — its own **tappable** control (text-button / link style), labelled from the catalog. Not buried inside the status sentence, and not a second screen. For most statuses the label is “Try again” and the tap starts a new attempt on this device. For **not set up**, the label opens setup instead (below) — another attempt cannot succeed until a method is in use.

Trying again is allowed only when nothing has been adopted yet; a result that is already showing stays.

Every failure uses that same layout. The status is one of these:

1. **Generic failure** — the transcript could not be made (English source: “Couldn't make a transcript.”). Optional detail may refine the case; when on-device is in use but not ready, decision 007 supplies detail English source: “On this device isn't ready yet.” Action remains Try again.
2. **Not set up** — **no** transcription method is in use (English source: “Transcription isn't set up.”). Do not use this status when a method is selected but temporarily unable. The action is a setup link (not Try again): it opens decision 005’s transcription selector / setup. Decision 009 adds a stronger one-shot call-to-action the first time this status appears on a voice message; afterwards this quiet status + setup link remains.
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

- Title is at most four words and at most 35 characters (same limits as the 006 / 007 prompts). A longer title is truncated on a character boundary to 35 before it is shown or sent; excess words are dropped from the end if needed to meet the four-word cap after that.
- Summary is at most four short sentences, and at most 20 words, and at most 160 characters, and is not a copy of the transcript. Longer summaries are truncated to those caps before show/send (character cap first, then word / sentence caps as needed).
- Transcript is the full text on the device that produced or adopted it. When that transcript is published in the companion notice, it may be shortened further for event size (below).

#### Room context (every call)

The service needs enough surrounding conversation to place what was said. Every call includes, built by the caller before invoking the service:

1. **Reply target** — if this voice message is a reply, that parent message as one context item (and a flag that it is the reply target).
2. **Recent messages** — the five most recent messages **before** this voice message, in the same timeline the voice message sits in (the room's main timeline, or the thread if the voice message is in a thread). Oldest of the five first, then toward the voice message. Fewer than five is fine when the timeline is shorter.

If the reply target is already among those five, it still appears once as content; the reply-target flag is what marks the relation. This voice message itself is not included in the five.

Each context item is **text only**. Never send prior messages' media bytes (audio, image, video, file) to the service. Per item:

| Kind                                                                    | What is sent                                                                                                                                                   |
| ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ordinary text message                                                   | The message body text                                                                                                                                          |
| Voice message that already has an adopted transcript (local or trusted) | That transcript text (and title/summary if adopted — enough that the service sees the spoken words as text)                                                    |
| Voice message without an adopted transcript                             | A short kind label only, e.g. that it is a voice message — not the audio                                                                                       |
| Image (and other non-text media: video, file, sticker, location, etc.)  | A short kind label only, e.g. that it is an image — not the media. If the event has a text caption, that caption text may be included; the media itself is not |
| Redacted / unavailable                                                  | A short kind label that it is unavailable — not an attempt to recover content                                                                                  |

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

Title and summary truncation to the limits above, and companion-notice transcript truncation for event size (below), sit outside the service, in the one place that shows and sends the result. All implementations share them. Mapping a service failure to one of the statuses above also sits outside the engine, in that same place.

The mock does not read the audio or the context. On a successful call it returns a fixed title, summary, and transcript. The fixed text says that it is sample text, not what was spoken. **Every third call fails** with a generic failure so the error UI can be exercised without a real engine. The mock can also return any of the other statuses under the same failure contract (useful for previews and tests).

**Interim only.** The mock is valid only until a real listener exists for the method in use:

- **Custom server** → decision 006 replaces the mock on that path.
- **On this device** → decision 007 replaces the mock on that path.
- **Flim** → decision 008 replaces the mock on that path.

Once that listener is in the replay, the composition root selects it for that method — never the mock. With no method in use, show not-set-up; do not call the mock as a stand-in. Until a path’s real listener exists, that method stays unavailable (or, only in the gap before 006–008 land, the mock may still serve real rooms for whatever path is wired). After 006–008, the mock remains available for tests and previews only.

A later decision replaces the mock on a path by swapping which implementation is constructed. It does not edit the bubble, the race, the failure UI, context gathering, or the notice. Credentials, a model, quota accounting, or a network client that the engine needs live inside that implementation. A new user-visible behaviour is a new decision, not a change to these callers.

The race below is per device. Only the device that sent the voice message publishes a brief to the other people in the room. If a recipient produces a result, it stays on that device.

### Sending

Flim sends the voice message immediately. It does not wait for the service.

Every voice message Flim sends carries a marker that a brief from this sender may follow. The marker is set even when the brief is not ready, and even if the brief later fails. Receivers use it only to decide whether to wait.

The sending device starts the service immediately. When the result returns, that device adopts it if it has not already adopted one, and publishes one companion notice if the room does not already have a trusted brief for this voice message. Publishing retries the same adopted result; it does not call the service again. If the notice would be too large for Matrix, shorten only the transcript in that notice (see Protocol) — do not drop the publish, and do not map size into a bubble failure status. That notice is how the original sender's title and summary reach the other people in the room. When the service fails, nothing is adopted, no notice is published, and the bubble shows the matching error.

Another signed-in device of the same account did not send the message. It follows the receive path and does not publish.

### Receiving

Auto-start of a brief on a recipient device applies only to a **newly arriving** voice message.

**Newly arriving** means this device is processing that voice-message event for the first time as a live or catch-up sync (including messages that landed while the app was closed or offline). It does **not** mean:

- opening a room whose timeline already holds the event,
- scrolling or paginating older history into view,
- reloading a room, or any other path that only puts an already-known past event on screen.

Those historical voice messages stay as the player alone unless a trusted companion notice is already available to adopt (below), or the user later taps Try again after a failure from an attempt that had already started. Showing them on screen never starts the wait or the service.

When a voice message **newly arrives**:

- **No Flim marker.** Start the service immediately.
- **Flim marker.** Wait 15 seconds for a trusted brief from the voice message's sender.
  - If that brief arrives during the wait, adopt it and do not start the service.
  - If the wait ends with nothing adopted, start the service.

The first result to become ready on this device is adopted. That includes a trusted brief that arrives while the local service is still running: the brief is ready first, the local result is discarded when it later finishes. A trusted brief that arrives after a local result was already adopted is discarded. Its text is not shown. Flim still does not draw that notice as its own row.

A recipient does not publish. The result they produce is for themselves on that device, and the other people in the room do not receive it.

**Trusted** means: the notice is the companion defined below (`msgtype` `m.notice` with a `me.flim.voice_summary` content block), its sender is the same user as the voice message's sender, and it links to this voice message as follows:

- **Main timeline** — `m.relates_to.rel_type` is `me.flim.voice_summary` and `m.relates_to.event_id` is this voice message.
- **Thread** — `m.relates_to.rel_type` is `m.thread` (so the notice stays in the thread), and `m.relates_to.m.in_reply_to.event_id` is this voice message.

Anything else with the marker stays an ordinary message — it is not attached to the player, and it is not hidden. The marker must not be a way for someone else in the room to overwrite a transcript or to hide a message.

If a trusted companion is discovered for a historical voice message (one that did not newly arrive on this device), adopt it when found. Do not start the 15-second wait or the local service solely because that message is on screen.

An adopted result is kept on device for that voice message, so a later launch does not start another attempt and does not swap in a late notice.

### On app launch (after a close or kill)

Incomplete work is remembered per voice message on this device. When the app starts again (or returns to a session that can run the service), unfinished briefs are handled as follows.

**Received** (this device did not send the voice message — including another device of the same account):

- Resume only work that **already started** for a newly arriving message (a wait in progress, or a service call that had begun): continue that wait or service. No 15-second wait again if that wait already completed before the close; if the wait was still running, finish the remaining wait, then continue as on receive.
- Do **not** scan the timeline or start new briefs for historical voice messages that never began an attempt.
- Voice messages that newly arrive during catch-up after launch follow **Receiving** as usual.
- A received result stays on this device. Relaunch never publishes for a received message.

**Sent** (this device sent the voice message):

- Resume only while the voice message is **younger than 10 minutes** (from its send time).
  - No adopted result and no trusted brief in the room → start the service immediately (no 15-second wait).
  - Adopted result not yet published → publish that result; do not call the service again.
- **10 minutes or older:** do not auto-start or resume the service, and do not publish a late companion notice. Recipients should not get a Flim title/summary in the room long after they already heard the audio. The bubble may show the quiet failure (or stay without a brief); the user can still tap Try again for a **local** result on this device only — that retry must not publish once the send is past 10 minutes.

The 10-minute limit applies only to **outgoing** auto-resume and to **publishing**. It does not limit received local briefs that already started for a newly arriving message.

### What other clients see

They see a normal voice message, then a notice that replies to it. The notice's visible text is only:

1. a short line that names Flim Messenger as the summary's source
2. a blank line
3. the title
4. a blank line
5. the summary

That leading line is the sender's Flim-catalog string (decision 002). English source: **"Summary by Flim Messenger"**. It comes first so other clients do not read the (often first-person) title and summary as the sender's own typed words. The full transcript is not in the visible text.

The notice is an `m.notice`, so default push rules do not raise a second notification for it. The voice message still notifies as it does today. Clients that have turned notice suppression off may notify. The notice is its own timeline event, so other clients that count every event will count it.

Flim does not draw a trusted companion notice as its own timeline row, whether or not its text was the result Flim adopted. On Flim, a **trusted** companion never counts toward the client-side unread state (room list badge, “has unread”, or advancing the read marker solely because that notice arrived). There is nothing visible in the timeline to read for that event. The voice message itself still follows ordinary unread rules. Untrusted events that happen to carry the marker stay ordinary messages and keep ordinary unread behaviour.

If the voice message is in a thread, the companion notice is in that same thread (see Protocol). Matrix allows only one `rel_type` on an event, so a threaded notice cannot also use the custom voice-summary relation.

When the sender redacts the voice message, Flim also redacts the companion notice they published for it, and drops any result stored on device for that message. The transcript should not remain in the room after the audio is removed.

### Protocol

Namespace is the product id from decision 000a: `me.flim`. Custom keys live in the event `content`, which is encrypted in encrypted rooms. They are not placed in `unsigned`.

**Voice message** (Flim-sent), in addition to the ordinary voice-message content:

```json
"me.flim.voice": { "v": 1 }
```

Presence is the whole signal. No brief is stored on the audio event.

**Companion notice** — shared fields on every companion:

```json
{
  "msgtype": "m.notice",
  "body": "Summary by Flim Messenger\n\n<title>\n\n<summary>",
  "me.flim.voice_summary": {
    "v": 1,
    "title": "<title>",
    "summary": "<summary>",
    "transcript": "<transcript>"
  }
}
```

`body` is plain text. There is no `formatted_body`. The transcript exists only under `me.flim.voice_summary`. The leading attribution in `body` is the sender's localized catalog string; the English source above is the fallback shape. Do not put a second attribution at the end.

**Companion notice size.** Matrix events are limited to 64 KiB; encryption adds overhead. Before publish, if the companion’s cleartext `content` would exceed **48 KiB**, shorten only `me.flim.voice_summary.transcript` from the end and append an ellipsis (`…`) so cleartext `content` stays within 48 KiB. That budget leaves headroom under the 64 KiB event limit for encryption and transport. Keep the `transcript` field; do not omit it for size. Prefer at least the ellipsis alone over removing the field. Title, summary, and `body` stay at the caps above and are not shortened further for size.

This truncation applies to the **published notice only**. The sending device keeps the full adopted transcript on the voice-message block. A recipient who adopts from the notice uses the (possibly truncated) transcript stored there. Size-based truncation is not a failure status and must not block sending the notice.

**`m.relates_to` — choose exactly one shape:**

1. **Voice message is not in a thread** (main timeline). Use the custom relation; `event_id` and `m.in_reply_to` both point at the voice message:

```json
"m.relates_to": {
  "rel_type": "me.flim.voice_summary",
  "event_id": "$voiceEventId",
  "m.in_reply_to": { "event_id": "$voiceEventId" }
}
```

2. **Voice message is in a thread.** Use `m.thread` so the notice appears in that thread. `event_id` is the thread root (same as the voice message's thread). `m.in_reply_to` points at the voice message — that is how Flim binds the notice to the audio when the custom `rel_type` cannot be used:

```json
"m.relates_to": {
  "rel_type": "m.thread",
  "event_id": "$threadRootEventId",
  "m.in_reply_to": { "event_id": "$voiceEventId" },
  "is_falling_back": false
}
```

Do not combine both `rel_type` values on one event. Do not invent a second relation field.

One notice per voice message. A later correction is not sent: a result that finishes later is discarded, not edited in.

### After the app leaves the foreground

Producing the brief and publishing the notice are part of handling the voice message, not part of keeping the room open.

Upstream already hands outgoing messages to a send queue, and when the app enters the background it starts a limited background interval before pausing client services. This decision requires the same kind of attempt for an in-flight brief: keep producing the result (and, on the sending device, publish the notice while still within the 10-minute window), across leaving the conversation and across the app going to the background.

The scheduler is not chosen here (extending that existing interval, a later refresh task, or something else). Closing or backgrounding the app must not be treated as a reason to drop an in-flight attempt early. If the system ends the process before work finishes, the next launch follows **On app launch** above: received work resumes; outgoing work resumes only under 10 minutes from send. If an outgoing notice never published in time, there is no late notice — recipients use their 15-second wait and their own on-device result.

## Platform notes

- iOS: the Rust SDK's typed timeline content drops unknown content keys. Reading `me.flim.voice` and `me.flim.voice_summary` needs the original event JSON (already exposed on timeline debug info) or a small SDK pass-through. The typed voice-message model is not enough.
- iOS: title/summary character limits mean extended grapheme clusters, so an emoji counts as one.
- iOS: status, the leading companion-notice attribution (“Summary by Flim Messenger”), each failure status above, and “Try again” strings are Flim catalog keys (decision 002), not `Localizable.strings` / `Untranslated.strings`. English source for Flim's allowance status must state that a larger transcription-service subscription is needed.
- iOS background: the existing pieces to hang the attempt on are the SDK send queue and `AppCoordinator`'s `beginBackgroundTask` when entering background. Which of those, or a new task, actually runs the service is an implementation choice.
- iOS: one protocol for the service, including room context on every call and a typed failure for each status above. This decision adds the mock as the interim implementation (success with sample text, every third call a generic failure, the other statuses available for previews/tests). The composition root is the only place that names the mock. After 006 / 007 / 008, those listeners are selected for their methods; the mock is then for tests and previews only.
- iOS: context items are built from timeline / local adopted briefs before the service call; do not download prior media solely to feed the service.
- Android: same product rules (bubble, race, companion notice, newly-arriving trigger). Map storage, background work, and event JSON access to the Android SDK.

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
- 2026-10-14: recipient auto-brief only for newly arriving voice messages (live/catch-up); opening or scrolling history does not start the wait or the service; relaunch resumes only in-flight received work; trusted companions for historical messages may still be adopted when found
- 2026-10-14: companion `body` for other clients starts with “Summary by Flim Messenger”, then title and summary; trailing “Made with Flim” dropped
- 2026-10-14: platforms include android (with ios)
- 2026-10-14: companion `m.relates_to` — custom `voice_summary` on the main timeline; `m.thread` + `m.in_reply_to` the voice event when the voice message is in a thread (one `rel_type` only); Trusted updated to match
- 2026-10-14: Not set up = no method in use only; on-device not-ready uses Generic failure + detail (decision 007)
- 2026-10-14: mock interim only until 006 / 007 / 008 replace it per path; no separate 010 decision
- 2026-10-14: title/summary limits match 006/007 prompts (title ≤4 words / 35 chars; summary ≤4 short sentences / 20 words / 160 chars)
- 2026-10-14: companion notice cleartext content capped at 48 KiB — oversized `transcript` truncated with `…`; sender keeps full transcript locally; size is not a failure status
- 2026-10-14: trusted companion excluded from Flim client-side unread (room badge / has-unread / read marker); voice message unread unchanged; other clients may still count the event
- 2026-10-14: brief UI — title always; body toggles summary (default) ↔ full transcript on tap; text selectable/copyable; long-press select does not toggle
- 2026-10-14: chevron-only disclosure always visible (down when summary, up when transcript); no Show/Hide text label
