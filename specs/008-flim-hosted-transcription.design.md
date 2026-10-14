---
decision: 008
slug: flim-hosted-transcription
created: 2026-10-13
---

# 008 — Flim hosted transcription (design)

Companion to `008-flim-hosted-transcription.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Decision 004 defines the in-app service contract (audio + room context → title, summary, transcript, or a typed failure) and ships a mock. Decision 005 lets the user finish Flim setup (email, code, plan) and make that method in use. Decisions 006 and 007 are the listeners for custom server and on-device. This decision is the real listener for **Flim**, and the product shape of Flim's EU-hosted service.

Callers, the bubble, the race, the companion notice, and the failure statuses stay as 004 specifies. When Flim is in use, the composition root selects this implementation instead of the mock.

Unlike custom server (006), the prompts live on Flim's service, not in the app. The app sends the voice message and the room context from decision 004, and the Flim session from decision 005.

Decision 005 owns the setup screens (email, code, plan, account). This decision owns what Flim's service does for those steps and for producing the brief.

## Product design

### Account and plan (service)

The account is an **email address**, not a Matrix user. One account serves every Matrix session on the device that holds the Flim session. Signing out of Matrix does not sign out of Flim.

#### Email and one-time code

1. The app submits one email address. The service sends a one-time code to that address.
2. The app submits the code. A new address **creates** the account; an existing address **signs in**. Either way the service returns a **session** the app stores (decision 005).
3. The user may ask for a new code. An invalid or expired code is refused in plain language the app can show on the code step.
4. A session can be ended (sign-out). After that, brief calls with that session are **sign-in rejected**.

Leaving setup before a plan is attached keeps the account and session; the app returns to the plan step (decision 005). Flim is not the method in use until a plan is attached.

#### Plan — promo code (ships with this decision)

Transcription needs an **active plan** with remaining allowance.

- The app submits a promo code while signed in. A code the service accepts **attaches a free plan** to that account.
- A code the service refuses leaves the account without that plan; the app stays on the plan step and shows why, in plain language.
- A second device that signs in to an account that already has a plan does **not** redeem another promo for the same plan; it reuses the plan that is already there.
- A free promo plan is capped at **100 voice-message briefs per calendar day** (UTC) for that account. A request that would go over the cap is refused as **allowance used up** (Flim wording from decision 004). The day counter resets at the next UTC midnight.

#### Plan — in-app subscription (placeholder)

Decision 005 shows subscription and restore on the plan step and account screen as **disabled**, with copy that they are **coming later**.

This decision reserves the service side of that path: when a later decision ships store billing, the app will send a platform purchase (or restore) for the signed-in account, and the service will attach or refresh a paid plan on that account — including restore onto the account they are signed in to. Until then, the service does **not** verify App Store or Play purchases. Promo remains the way to get a plan for the first ship.

The bubble never opens a store. Allowance exhaustion uses decision 004's Flim allowance wording; the plan step and the account screen are where the user applies a promo (or, later, subscribes).

### When the brief listener runs

Only while decision 005's Flim method is in use. The saved Flim session is the only credential this listener sends. Cortecs keys, model names, and prompts are not configured in the app.

### What the app sends

One HTTPS call to Flim's service per decision 004 attempt:

- The voice message audio
- The room context from decision 004 (reply target and recent text items — never prior media bytes)
- The Flim session (so the service can check the account and plan)

The app does not choose transcription or summary models. It does not send prompts.

### What the app gets back

On success: a title, a summary, and a transcript — the same three fields decision 004 already expects. Title may be longer than 50 characters here; truncation to 50 characters stays outside this listener, as decision 004 already requires.

On failure: a reply the listener maps to decision 004's statuses (below). One attempt is the whole call; nothing is adopted unless all three fields arrive in a usable shape.

### What Flim's service does

1. Authenticates the session. Rejected or missing session → **sign-in rejected**.
2. Checks the account has an active plan with remaining allowance. Empty allowance → **allowance used up** (Flim wording from decision 004).
3. Transcribes the audio with Cortecs, model **faster-whisper-large-v3**, with EU-native routing so inference stays with EU-based providers.
4. Asks Cortecs chat model **ministral-8b:latest** (same EU-native routing) for a title and summary, using prompts the service holds. Prefer a **low-latency** Cortecs provider for that model; today **Mistral**'s own serving of it is the fast path — pin or prefer that provider when Cortecs allows. Those prompts follow the same product rules as decision 006's embedded prompts: short title, short summary, same language as the transcript, first-person voice, room context only to place what was said, JSON with `title` and `summary`. Exact prompt wording is owned by the service and may differ from 006's copy.
5. Returns title, summary, and transcript to the app. Empty transcript → **unusable reply**. Title/summary JSON that cannot be parsed → **unusable reply**.

Audio and room context leave the phone only for this service and its EU inference path. The service does not publish to the room and does not store briefs for callers; decision 004's callers own show/send.

**Retention:** the service keeps what the account needs (email, session material, plan and allowance, promo redemptions, and later store-purchase references). It does **not** retain voice audio, transcripts, titles, summaries, or room context after the request finishes. Those leave only for the live Cortecs calls.

### Mapping replies to decision 004 failures

| Situation                                                                 | Status (004)                     |
| ------------------------------------------------------------------------- | -------------------------------- |
| No HTTP response, timeout, or connection drop                             | No reply                         |
| HTTP 401 / 403 (session missing or rejected)                              | Sign-in rejected                 |
| HTTP 429 that clearly means plan allowance exhausted                      | Allowance used up (Flim wording) |
| HTTP 429 otherwise, or a throttle that is not readable as empty allowance | Too many requests                |
| HTTP 400 / 404 / 422 that points at request shape or setup                | Setup is wrong                   |
| Other 4xx / 5xx, or a server-side failure body                            | Service fault                    |
| Success body that is not a title, summary, and transcript                 | Unusable reply                   |

Prefer the more specific row when more than one could apply. Do not invent new failure statuses here.

### Out of band

This listener does not publish to the room, does not show UI, and does not store results. Decision 004's callers own those. Cancellation of an in-flight attempt cancels the outstanding HTTP call to Flim.

### Hosting shape (product claim)

Flim's service runs in the **EU** (UpCloud, Finland). Inference for transcription and summary is Cortecs (European company) with **EU-native** provider routing. A single Kubernetes worker is enough for the first ship; the operator may add workers later when zero-downtime rolling upgrades across nodes are needed. Brief interruptions on a one-node deploy or node replacement are acceptable until then.

## Platform notes

- Both apps: selected at the composition root when Flim is the method in use; conforms to the same service protocol as the mock (decision 004) and the other listeners.
- iOS: the Flim session comes from the Keychain entry decision 005 already requires; do not cache it in preferences.
- Android: the Flim session comes from the platform credential store decision 005 already requires; do not cache it in preferences.
- Web: deferred.
- Flim's service (implementation of this host, not app UI): **NestJS (TypeScript)**; **UpCloud Managed Kubernetes** Development control plane; **one** smallest allowed worker (2 GB minimum); **smallest** UpCloud Managed PostgreSQL; Cortecs OpenAI-compatible API for **faster-whisper-large-v3** (transcribe) and **ministral-8b:latest** (title/summary), with `eu_native` set so routing stays EU. For Ministral, prefer a low-latency provider — currently Mistral. Use Resend for email. Reasonable rate limiting.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-14: session 401/403 maps to Sign-in rejected (004 rename); drop key-as-session wording
- 2026-10-14: summary model locked to ministral-8b:latest; prefer low-latency Cortecs provider (currently Mistral)
- 2026-10-14: account/plan — email OTP, promo plan, IAP placeholder; retention keeps account state only, not audio or briefs
- 2026-10-14: promo free plan 100 briefs/day (UTC); IAP aligned with 005 disabled + coming-later UI
