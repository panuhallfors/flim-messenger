---
id: 010
slug: flim-push-gateway
title: Flim push gateway
status: proposed
created: 2026-10-14
amends: []
platforms: [ios, android]
---

# 010 — Flim push gateway

## Intent

With Flim’s own app identity (decision 000a), the user still receives Matrix push notifications on a locked phone. Pushes go through **Flim’s EU push gateway**, not Element’s `matrix.org` gateway, which only delivers for Element’s app ids.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or looking at where push traffic is sent).

- [ ] With notifications enabled, a new message that should notify reaches a locked phone as a system push for the Flim app (not only while the app is open).
- [ ] The app registers its pusher against `https://push.flim.me`, not `matrix.org`.
- [ ] iOS and Android each receive pushes under Flim’s own app identity (bundle / application id family from decision 000a), not Element’s.

## Non-goals

- Replacing or rewriting the push gateway software — use [Sygnal](https://github.com/element-hq/sygnal) (Element HQ’s reference gateway), obtained and deployed by the meta `push-gateway/` tree.
- Changing upstream notification UI, mute / push-rule behaviour, or the Notification Service Extension’s job of fetching and decrypting event content.
- OIDC / MAS domains, privacy / terms links, analytics, Sentry, or bug-report endpoints (separate from this decision).
- Operating a Matrix homeserver; this decision is only the push gateway homeservers call.
- Zero-downtime multi-node Sygnal on day one (same first-ship bar as decision 008’s single worker).

## Notes

Upstream Element X defaults `pushGatewayBaseURL` to `https://matrix.org`. That Sygnal holds Element’s APNs / FCM credentials for Element’s `app_id`s only. After 000a’s identifiers, that path cannot deliver to Flim.

**Human-owned:** DNS for `push.flim.me`, Apple APNs and Google FCM credential creation, cluster access secrets, and a locked-phone smoke-check. **Implementation of this decision:** a meta-root **`push-gateway/`** sub-project (sibling to `ios/`; listed in `AGENTS.md` Layout) that obtains Sygnal, deploys it to the shared UpCloud namespace, and sets up TLS for that host, plus pointing each app at `https://push.flim.me`. Product shape is in the design.

Design (replay-canonical with this decision): [`010-flim-push-gateway.design.md`](010-flim-push-gateway.design.md)

## Verification

### ios

- Tests: none yet (gateway URL `https://push.flim.me` and `app_id` wiring are lockable in unit tests; end-to-end push needs a real device / APNs)
- Agent UI: none
- Meta: `push-gateway/` deploy tree is part of this decision’s implementation (not an app submodule pin)

### android

- deferred (same product design; FCM end-to-end when Android ships)

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-14: proposed
- 2026-10-14: gateway host locked to `push.flim.me`
- 2026-10-14: Sygnal deploy + app URL wiring are implementation (`push-gateway/` in meta); humans only credentials / DNS / smoke-check
- 2026-10-14: TLS on deploy; DNS manual; `AGENTS.md` Layout lists `push-gateway/`
