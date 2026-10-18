---
id: 006
slug: custom-server-transcription
title: Custom server transcription
status:
  ios: proposed
  android: proposed
created: 2026-10-13
amends: [004]
platforms: [ios, android]
---

# 006 — Custom server transcription

## Intent

When the user has finished custom-server setup (decision 005) and that method is in use, Flim produces the title, summary, and transcript by talking to that server over an OpenAI-compatible API. The app holds the prompts. That path no longer uses decision 004's mock.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] With a custom server as the method in use, a voice message's title, summary, and transcript come from that server — not from the sample mock text.
- [ ] The same brief presentation, race, room notice, and failure statuses from decision 004 still apply; only the source of the words changes on this path.
- [ ] A server that refuses the key, is unreachable, returns a bad reply, or reports an empty allowance shows the matching failure from decision 004.
- [ ] Flim and on-device methods are unchanged by this decision: they use their own listeners when 007 / 008 are in the replay. Until then those methods stay unavailable — they do not keep using the mock once a real listener exists for a path.

## Non-goals

- The settings UI that collects addresses, models, and the key(s) (decision 005).
- Hosting or shipping a server. The user names one.
- The hosted Flim transcription service and its account, plan, or prompts.
- On-device transcription and summarisation.
- Changing decision 004's bubble, wait, race, companion notice, or failure status list.
- Letting the user edit the embedded prompts in settings.
- A language picker or other per-call transcription options beyond model, key(s), and addresses.

## Notes

Decision 005 chooses the method and stores the custom-server fields. This decision is the listener for that method: how the app calls the server and turns the replies into a brief.

The product design is the same on iOS and Android.

New user-facing strings belong to the Flim catalog (decision 002). Prompt text that only the server sees is not a catalog string.

Design (replay-canonical with this decision): [`006-custom-server-transcription.design.md`](006-custom-server-transcription.design.md)

## Verification

### ios

- Tests: none yet (request shaping, reply parsing, and status mapping are lockable with unit tests against a fake OpenAI-compatible host)
- Agent UI: none

### android

- Tests: none yet (request shaping, reply parsing, and status mapping are lockable with unit tests against a fake OpenAI-compatible host)
- Agent UI: none

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-13: proposed
- 2026-10-13: design locks base-URL normalisation (including pasted full paths), https only, no language field, prompt-only JSON for title/summary
- 2026-10-13: design embeds adapted prior-Flim title/summary prompts (same language as transcript)
- 2026-10-13: Android implements this listener as well as iOS
- 2026-10-14: platforms = ios, android
- 2026-10-14: per-address keys (transcription vs summary); no cross-host key send
- 2026-10-14: other methods stay unavailable until 007 / 008 — no ongoing mock on those paths
- 2026-10-14: listener also powers 005’s custom-server setup check (bundled clip)
