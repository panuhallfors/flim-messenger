---
decision: 006
slug: custom-server-transcription
created: 2026-10-13
---

# 006 — Custom server transcription (design)

Companion to `006-custom-server-transcription.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Decision 004 defines the in-app service contract (audio + room context → title, summary, transcript, or a typed failure) and ships a mock. Decision 005 lets the user finish a custom-server setup and make that the method in use. This decision is the real listener for that method.

Callers, the bubble, the race, the companion notice, and the failure statuses stay as 004 specifies. When custom server is in use, the composition root selects this implementation instead of the mock. Flim is decision 008; on-device is decision 007. Each of those, like this decision, replaces the mock on its path when it is in the replay — the mock is only interim until then (decision 004).

## Product design

### When it runs

Only while decision 005's custom-server method is in use. The saved fields from that setup are the only credentials and endpoints this listener uses:

- transcription base address and model name
- summary base address and model name
- transcription API key
- summary API key (may be the same secret as the transcription key when the user chose “same key” or entered one key for a shared host)

No other method's credentials are sent. Each key is sent **only** as a bearer token to **its own** address: the transcription key to the transcription address, the summary key to the summary address. Never send the transcription key to the summary host (or the reverse) unless those two saved key values are the same secret the user intentionally stored for both.

### Protocol (OpenAI-compatible)

The user's server speaks an OpenAI-compatible HTTP API. Two steps, in order. One attempt in decision 004's sense is both steps; if either fails, nothing is adopted.

#### 1. Transcription

`POST {transcription address}/v1/audio/transcriptions`

- Multipart form: the voice message audio as `file`, the saved model name as `model`, and `response_format=json` (or `text` if the host only returns plain text — the app accepts either a JSON body with a `text` field or a plain-text body as the transcript).
- Header: `Authorization: Bearer <transcription key>`.
- Success: a non-empty transcript string. Empty transcript is an **unusable reply**.

#### 2. Title and summary

`POST {summary address}/v1/chat/completions`

- JSON body: the saved summary model name, and a short chat that includes Flim's embedded prompts, the transcript, and the room context from decision 004.
- Header: `Authorization: Bearer <summary key>`.
- The app asks for a structured reply the model can return as the assistant message content: JSON with `title` and `summary` string fields. A reply that is not parseable as that shape, or that is missing either field, is an **unusable reply**.
- Title or summary may exceed decision 004’s caps here; truncation to those caps (title ≤4 words / 35 characters; summary ≤4 short sentences / 20 words / 160 characters) stays outside this listener, as 004 already requires.

The two addresses may be the same host. Path suffixes above are relative to the base the user entered in settings (the base is the origin the user typed — e.g. `https://example.com` or `https://example.com/v1` — see normalisation below).

#### Address normalisation

Settings expect an API **base**, not a full endpoint. Before building paths:

1. Trim trailing slashes.
2. If the value ends with `/audio/transcriptions` or `/chat/completions` (with or without a preceding `/v1`), strip that suffix so a pasted full URL still becomes a base.
3. If the base already ends with `/v1`, append `/audio/transcriptions` or `/chat/completions`.
4. Otherwise append `/v1/audio/transcriptions` or `/v1/chat/completions`.

Do not invent other path variants. A host that needs a different layout is out of scope; the user picks a compatible server.

Addresses must use `https`. An `http` base is **setup is wrong** (do not call the server).

#### Language

The transcription request does not send a `language` field. The server uses its own default or auto-detect. A user-facing language setting is out of this decision.

### Embedded prompts

The prompts live in the app, the same wording on every platform (English source below). The user cannot edit them in settings. Flim's hosted service (a later decision) may use different prompts; this listener does not share a network prompt store.

These are adapted from the previous Flim generation: same length and tone rules, first-person voice, empty strings when nothing fits — but the language is “same as the transcript,” not a language code, and room context is included. Provider-specific safety blocks (e.g. Gemini) are not sent; an OpenAI-compatible chat call has no equivalent field here.

The app does **not** send `response_format: json_object` (or equivalent). Structured output is requested only in the system prompt. A non-parseable assistant message, or one missing `title` or `summary`, is an **unusable reply**. Empty-string `title` or `summary` is allowed when the model cannot find one (still a usable reply). Decision 004 still truncates a longer title or summary to its caps before show/send.

Changing the product intent of these prompts (limits, voice, language rule, or asking for more than title + summary) needs a new decision.

#### System message (exact)

```
Below is a transcription of a voice message from a person to another person, with optional surrounding room context. Please create a title (maximum four words, maximum 35 characters), and a short summary (the summary should be maximum four short sentences, and maximum 20 words, and maximum 160 characters) about what is covered in the message. Write both the title and the summary in the same language as the voice message transcript. Use the same tone of voice as the sender has in the original voice message; if the original message is informal, then also write informally. Do not refer to who is the one sending the message (example: instead of saying 'The speaker likes roses', just write 'I like roses'). Only capitalize the first letter of the title (apart from names of course). If you cannot find the title, just write an empty string. If you cannot find the summary, just write an empty string. Use the room context only to place what was said; do not invent spoken words that are not in the transcript. Reply with only a JSON object with string fields "title" and "summary", and nothing else.
```

#### User message (exact shape)

Built by the app each call, in this order:

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

### Mapping replies to decision 004 failures

| Situation | Status (004) |
|-----------|----------------|
| No HTTP response, timeout, or connection drop | No reply |
| HTTP 401 / 403 | Sign-in rejected |
| HTTP 429 that clearly means quota / allowance exhausted | Allowance used up (custom-server wording) |
| HTTP 429 otherwise, or a throttle that is not readable as empty allowance | Too many requests |
| HTTP 400 / 404 / 422 that points at model, path, or request shape | Setup is wrong |
| Other 4xx / 5xx, or a JSON error body that is a server-side failure | Service fault |
| Success body that is not a transcript, or not `{title, summary}` | Unusable reply |

Prefer the more specific row when more than one could apply. Do not invent new failure statuses here.

### Out of band

This listener does not publish to the room, does not show UI, and does not store results. Decision 004's callers own those. Cancellation of an in-flight attempt cancels outstanding HTTP work for both steps.

## Platform notes

- iOS: selected at the composition root when custom server is the method in use; conforms to the same service protocol as the mock (decision 004).
- iOS: the transcription and summary keys come from the Keychain entries decision 005 already requires; do not cache them in preferences.
- Android: selected at the composition root when custom server is the method in use; same service contract as iOS. The keys come from the credential-store entries decision 005 already requires; do not cache them in preferences.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: on-device listener is decision 007; this decision still does not implement it
- 2026-10-13: base URL may strip a pasted full OpenAI path; https required; no transcription language field; title/summary JSON via prompt only (no JSON-mode request field)
- 2026-10-13: http disallowed — https only
- 2026-10-13: embedded system/user prompts locked (adapted from prior Flim; language = same as transcript; no provider safety settings)
- 2026-10-13: Android implements this listener; Web stays deferred
- 2026-10-14: 401/403 maps to Sign-in rejected (004 rename)
- 2026-10-14: platforms = ios, android; web out of product scope
- 2026-10-14: separate transcription and summary keys; each bearer token only to its own address
- 2026-10-14: Flim / on-device stay unavailable until 007 / 008; mock not kept as stand-in on those paths
- 2026-10-14: truncation references point at 004’s prompt-matched caps (35 / 160), not 50
