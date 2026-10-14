---
decision: 007
slug: on-device-transcription
created: 2026-10-13
---

# 007 — On-device transcription (design)

Companion to `007-on-device-transcription.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Decision 004 defines the in-app service contract (audio + room context → title, summary, transcript, or a typed failure) and ships a mock. Decision 005 lets the user confirm **On this device** when the phone can transcribe and summarise there. This decision is that test, and the real listener for that method.

Callers, the bubble, the race, the companion notice, and the failure statuses stay as 004 specifies. When on-device is in use, the composition root selects this implementation instead of the mock. Flim and custom server keep their own listeners.

The words come from the operating system's own on-device speech and language models. The app does not carry those models. Today's device classes are in Platform notes so the gate is not written as if every recent phone could do both jobs.

## Product design

### When it runs

Only while decision 005's on-device method is in use. No network client, no API key, and no other method's credentials are used.

One attempt in decision 004's sense is both steps below. If either fails, nothing is adopted and no companion notice is sent.

### What "the device can" means

The choice can be confirmed only when the OS reports all of the following:

1. On-device speech-to-text for at least one language.
2. An on-device language model that can take the prompt below (a title and a summary from the transcript and room context). A model that only returns a fixed bullet summary, and cannot take this prompt, does not qualify.
3. The system assets for both are installed and ready, not still downloading.

The check is whatever the OS reports at runtime. A phone that can speak-to-text and cannot run that language model cannot confirm. A phone whose models are still downloading cannot confirm yet.

The row stays visible. The reason names the gap in ordinary language: no on-device speech, no on-device model that can title and summarise, the models are still preparing, or the on-device model does not support a language the phone can use. The app may start the system download. It does not download a model of its own, and it does not switch the user to Flim or a custom server.

Confirming while the check passes makes on-device the method in use. If a later check fails, the method stays selected, the settings row says it is not ready, and attempts fail as **Generic failure** (below) until the check passes again — not as **Not set up**, so decision 009 does not push the user back into setup. The user can switch to another method.

Use the best on-device speech engine the OS offers for the language. Do not refuse the method only because a newer, narrower speech engine is missing, when an older on-device engine and the language model are both ready. Decision 005 already tells the user the quality is lower than an external service.

### The two steps

#### 1. Transcription

The voice-message audio is passed to the OS speech engine. It stays on the device.

Success is a non-empty transcript. An empty transcript, an unsupported spoken language, or an engine error fails the attempt.

#### 2. Title and summary

The on-device language model receives the transcript and the room context from decision 004. That text stays on the device. The prompt is the same wording as decision 006: same length and tone rules, first person, the transcript's language, empty strings when nothing fits. The user cannot edit it. Changing those limits, the voice, the language rule, or asking for more than a title and a summary needs a new decision.

Where the OS can bind `title` and `summary` as two strings, use that. The system message is still sent as written. A reply that does not yield those two string fields is an **unusable reply**. An empty-string title or summary is allowed when the model cannot find one (still a usable reply). Decision 004 still truncates a longer title or summary to its caps (title ≤4 words / 35 characters; summary ≤4 short sentences / 20 words / 160 characters) before show/send.

A voice message the model cannot finish is a failed attempt. There is no partial brief and no overflow to a cloud model.

##### System message (exact)

```
Below is a transcription of a voice message from a person to another person, with optional surrounding room context. Please create a title (maximum four words, maximum 35 characters), and a short summary (the summary should be maximum four short sentences, and maximum 20 words, and maximum 160 characters) about what is covered in the message. Write both the title and the summary in the same language as the voice message transcript. Use the same tone of voice as the sender has in the original voice message; if the original message is informal, then also write informally. Do not refer to who is the one sending the message (example: instead of saying 'The speaker likes roses', just write 'I like roses'). Only capitalize the first letter of the title (apart from names of course). If you cannot find the title, just write an empty string. If you cannot find the summary, just write an empty string. Use the room context only to place what was said; do not invent spoken words that are not in the transcript. Reply with only a JSON object with string fields "title" and "summary", and nothing else.
```

##### User message (exact shape)

Built by the app each call, in this order. Same shape as decision 006.

```
Transcript:
<transcript>

Room context:
Reply target: <yes|no>
Sender: <display name or id>
Text: <context item text>

Sender: <display name or id>
Text: <context item text>
…
```

- `Transcript:` is always present (the non-empty string from step 1).
- `Room context:` lists the reply target first when there is one (with `Reply target: yes` on that item only), then the recent messages from decision 004, oldest of those first. Each item is one `Sender:` / `Text:` pair. Kind labels from decision 004 are the `Text` when there is no body. If there is no context, still include the `Room context:` heading with no items under it.

### Stays on the device

Audio, transcript, title, summary, and room context are not sent off the device for this method. A system API that can also call a cloud or private-cloud model is used only in its on-device mode. If that mode is unavailable, the phone cannot confirm this method.

### Mapping to decision 004 failures

This listener does not invent statuses. It does not produce the network ones: no reply, allowance used up, too many requests, or sign-in rejected.

**Not set up** is only for when **no** transcription method is in use. That case is owned by decision 004’s callers (and 009’s setup link), not by this listener while on-device remains the selected method.

| Situation | Status (004) |
|-----------|----------------|
| On-device is the method in use, but the phone cannot currently both transcribe and summarise on device (models still preparing, assets gone, on-device mode unavailable, or the readiness check fails when the attempt runs) | **Generic failure**, with detail English source: **“On this device isn't ready yet.”** Action stays **Try again** (not setup). |
| Unsupported spoken language, speech engine error, or model error while the device is otherwise ready | **Generic failure** (default status line; optional short human detail from the engine if useful and non-technical) |
| Empty transcript, or a reply that does not yield string fields `title` and `summary` | **Unusable reply** |

Prefer the more specific row when more than one could apply. Never map a selected-but-not-ready on-device method to **Not set up**.

### Out of band

This listener does not publish to the room, does not show UI, and does not store results. Decision 004's callers own those. Cancellation of an in-flight attempt cancels both steps.

## Platform notes

Runtime OS checks win over the lists below. The lists are the current device classes, so the product is not aimed at phones that cannot run a local model.

- iOS: speech is the system speech analyser (iOS 26 and later). Whether it is available, and for which languages, comes from the OS; language assets may still be downloading. The older dictation transcriber does not satisfy this decision on a phone that has no on-device language model.
- iOS: the language model is the on-device Apple foundation model, and only when Apple Intelligence is available and turned on for a supported language and region. That class today is iPhone 15 Pro, iPhone 15 Pro Max, iPhone 16 and later, iPhone Air, iPad mini (A17 Pro), and iPad with M1 or later. Guided generation may bind `title` and `summary`; the system message stays the exact text above. Private Cloud Compute and third-party models in the same framework are out of scope. A model that is still downloading is "still preparing", not a confirmed method.
- Android: speech is the system's on-device recogniser. A basic engine exists on most devices with Android 12 (API 31) and higher, for a limited set of languages. A stronger engine exists on a smaller set (Pixel 10 and Pixel 11 today). Use the stronger engine when the OS offers it; otherwise use the basic one. Do not refuse the method only because the stronger engine is absent.
- Android: the language model is the system on-device prompt model (Gemini Nano), which can take the system and user messages above. A bullet-only summariser does not qualify. Today that prompt model is a flagship set — Pixel 9 series and later, and other recent flagships on the OS list — checked at runtime, not hardcoded. Unlocked-bootloader devices are reported unavailable. Structured output may bind `title` and `summary`; the system message stays the exact text above. Model download belongs to the system; the app may start it; the method stays unconfirmed until the OS reports ready.
- Both: strings are Flim catalog keys (decision 002). The choice sits in decision 005's selector, which lists Flim, custom server, and on this device on both platforms.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: title/summary prompt locked to the same system message and user-message shape as decision 006; empty title or summary is usable; a bullet-only summariser does not qualify
- 2026-10-13: dropped the Android on-device-only settings exception; the full selector is decision 005 on both platforms
- 2026-10-14: network status name updated to Sign-in rejected (004 rename)
- 2026-10-14: platforms = ios, android
- 2026-10-14: selected-but-not-ready on-device maps to Generic failure (“On this device isn't ready yet.”) + Try again; Not set up only when no method is in use
- 2026-10-14: truncation references match 004 / 006 prompt caps
