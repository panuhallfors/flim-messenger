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

When the user has finished Flim setup (decision 005) and that method is in use, Flim produces the title, summary, and transcript by talking to Flim's own EU-hosted service. That service also owns the account behind that setup: email one-time codes, plans from a promo code, and a place for later in-app subscription purchases. That path no longer uses decision 004's mock.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] With Flim as the method in use, a voice message's title, summary, and transcript come from Flim's service — not from the sample mock text.
- [ ] The same brief presentation, race, room notice, and failure statuses from decision 004 still apply; only the source of the words changes on this path.
- [ ] A missing or rejected session, an empty allowance, an unreachable service, or a bad reply shows the matching failure from decision 004.
- [ ] The user can create or sign in to a Flim transcription account with an email and a one-time code sent to that address, as decision 005's setup expects.
- [ ] The user can attach a plan with a promo code the service accepts; a refused promo stays without a plan and can try again.
- [ ] A free promo plan allows at most 100 voice-message briefs per day; going over shows decision 004's allowance failure.
- [ ] In-app subscription and restore stay reserved for a later ship (decision 005 shows them disabled with coming-later copy). Completing a paid plan through the store is not required for this decision to be done.
- [ ] Custom server and on-device methods are unchanged by this decision.

## Non-goals

- The settings screens and copy for Flim setup (decision 005) — this decision is the service those screens call, and the brief listener.
- Custom-server and on-device listeners (decisions 006 and 007).
- Changing decision 004's bubble, wait, race, companion notice, or failure status list.
- Letting the user edit Flim's prompts in the app.
- Shipping App Store / Play billing verification in this decision (placeholder only).
- Shipping zero-downtime multi-node hosting on day one (one worker is enough until the operator adds nodes).

## Notes

Decision 005 chooses the method and keeps the Flim session on device. This decision is the listener for that method and the product shape of Flim's hosted service: accounts, plans, protocol, prompts, and where inference runs.

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
- 2026-10-14: acceptance covers email OTP account, promo plan, and IAP as a later placeholder; settings UI stays decision 005
- 2026-10-14: promo free plan capped at 100 briefs/day; IAP remains disabled-with-coming-later in 005
