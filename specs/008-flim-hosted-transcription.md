---
id: 008
slug: flim-hosted-transcription
title: Flim hosted transcription
status: proposed
created: 2026-10-13
amends: [004]
platforms: [ios, android]
---

# 008 — Flim hosted transcription

## Intent

When the user has finished Flim setup (decision 005) and that method is in use, Flim produces the title, summary, and transcript by talking to Flim's own EU-hosted service. That path no longer uses decision 004's mock.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] With Flim as the method in use, a voice message's title, summary, and transcript come from Flim's service — not from the sample mock text.
- [ ] The same brief presentation, race, room notice, and failure statuses from decision 004 still apply; only the source of the words changes on this path.
- [ ] A missing or rejected session, an empty allowance, an unreachable service, or a bad reply shows the matching failure from decision 004.
- [ ] Custom server and on-device methods are unchanged by this decision.

## Non-goals

- The settings UI that creates the Flim account and plan (decision 005).
- Custom-server and on-device listeners (decisions 006 and 007).
- Changing decision 004's bubble, wait, race, companion notice, or failure status list.
- Letting the user edit Flim's prompts in the app.
- Shipping zero-downtime multi-node hosting on day one (one worker is enough until the operator adds nodes).

## Notes

Decision 005 chooses the method and keeps the Flim session. This decision is the listener for that method and the product shape of Flim's hosted service (protocol, prompts, where inference runs).

The product design is the same on iOS and Android. Web stays deferred.

New user-facing strings belong to the Flim catalog (decision 002). Prompt text that only the service sees is not a catalog string.

Design (replay-canonical with this decision): [`008-flim-hosted-transcription.design.md`](008-flim-hosted-transcription.design.md)

## Verification

### ios

- Tests: none yet (request shaping, reply parsing, and status mapping are lockable with unit tests against a fake Flim host)
- Agent UI: none

### android

- Tests: none yet (request shaping, reply parsing, and status mapping are lockable with unit tests against a fake Flim host)
- Agent UI: none

### web

- deferred

## History

- 2026-10-13: proposed
