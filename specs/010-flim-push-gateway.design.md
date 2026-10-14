---
decision: 010
slug: flim-push-gateway
created: 2026-10-14
---

# 010 — Flim push gateway (design)

Companion to `010-flim-push-gateway.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Matrix homeservers do not talk to APNs or FCM directly. They POST to a [push gateway](https://spec.matrix.org/latest/push-gateway-api/#overview). Clients register a **pusher** with `app_id`, device `pushkey`, and the gateway’s `_matrix/push/v1/notify` URL. Element X uses `event_id_only` so the gateway sees room / event ids (and enough to wake the app), not message bodies; the app’s notification extension then loads and decrypts the event from the homeserver.

Decision 000a gives Flim its own bundle / application ids. Upstream’s default gateway (`matrix.org`) cannot deliver for those ids. Decision 008 already places Flim’s transcription service on **UpCloud Managed Kubernetes in Finland**. This decision adds Flim’s push gateway beside that service.

Reference implementation: [element-hq/sygnal](https://github.com/element-hq/sygnal) (moved from the former matrix-org location).

## Product design

### Where pushes go

- Flim’s clients register HTTP pushers whose gateway base URL is **`https://push.flim.me`**, not `https://matrix.org`.
- The notify path stays the Matrix standard: `https://push.flim.me/_matrix/push/v1/notify`.
- Payload format stays **event id only** (upstream Element X behaviour): the gateway does not receive message text; encrypted content is still resolved on device.
- `app_id` values follow the same family as decision 000a’s product id, with the platform suffix pattern Element X already uses (e.g. iOS debug / prod variants). Exact strings are platform translation.

### Hosting shape

- Run **Sygnal** in the **EU**, on the **same UpCloud Managed Kubernetes cluster and namespace** as decision 008’s Flim transcription service.
- One small Deployment is enough for the first ship; brief interruption on node replacement is acceptable until the operator adds capacity.
- Public name **`push.flim.me`**: **DNS is human**; **TLS termination is part of the deploy** (ingress + certificate issuance in the `push-gateway/` scripts, e.g. cert-manager / ACME once the name resolves).
- APNs auth material and FCM / Firebase credentials live only as cluster secrets (or equivalent), not in git.

### Meta sub-project (`push-gateway/`)

Gateway deploy is **implementation of this decision in the meta repo**, not a human checklist step and not code inside `ios/` / `android/`. Meta `AGENTS.md` lists this tree in Layout.

- Add a first-party tree **`push-gateway/`** at the meta root (sibling to `ios/` and `android/`). It is owned by this meta repository — not an Element app submodule.
- That tree obtains [element-hq/sygnal](https://github.com/element-hq/sygnal) (pinned image and/or build), holds Flim’s Sygnal config (`app_id`s for the 000a identity family, APNs/FCM app entries), Kubernetes manifests / deploy script, and publishes Sygnal into the shared UpCloud namespace so `https://push.flim.me/_matrix/push/v1/notify` is live.
- The same deploy path sets up **TLS** for `push.flim.me` (certificate request and ingress HTTPS). It assumes DNS already points at the cluster ingress; it does not register DNS records.
- Secrets (APNs key, FCM service account, cluster kubeconfig) are supplied out of band; the sub-project must not commit them.
- Exact tooling (shell, Helm, kustomize, cert-manager, etc.) is filled in when implementation begins.

### Apps

- iOS and Android set their push gateway base URL to **`https://push.flim.me`** as part of implementing this decision (same change on every platform).

### Privacy (product claim)

Push metadata for Flim users (that a given device should wake for a room / event, plus device push tokens at registration time) is processed by **Flim’s EU gateway**, not by Element’s `matrix.org` Sygnal. Message bodies are not part of the gateway payload under `event_id_only`.

### Human setup

Only DNS, credentials, cluster access, and a real-device smoke-check. Deploy (including TLS), and app wiring, are agent / implementation work under this decision.

1. **DNS** — point `push.flim.me` at the UpCloud ingress (A/AAAA or CNAME as the cluster requires). Manual; not automated by `push-gateway/`.
2. **Apple (iOS)** — create an APNs Auth Key (`.p8`), note Key ID and Team ID; topic is the iOS app’s bundle id family from 000a. Hand the material to the `push-gateway/` secret path (not git).
3. **Google (Android)** — create a Firebase / FCM project for Flim; hand the FCM credentials (API v1 service account preferred) to the `push-gateway/` secret path.
4. **Cluster access** — provide UpCloud Kubernetes credentials the deploy script needs (out of band).
5. **Smoke-check** — after deploy and app builds: notifications allowed, lock the phone, send a message that should notify, confirm a system push arrives for Flim.

### Out of band

This decision does not change how the notification UI looks, how push rules mute rooms, or how the NSE / Android equivalent enriches the notification. Homeserver choice is unchanged: any HS the user logs into must be able to reach Flim’s public gateway URL.

## Platform notes

### ios

- Upstream knob: `AppSettings.pushGatewayBaseURL` → `https://push.flim.me` (notify path appended as `_matrix/push/v1/notify`).
- `pusherAppID` already derives from `baseBundleIdentifier` + `.ios.dev` / `.ios.prod` — once 000a’s base id is `me.flim`, Sygnal’s `apps:` keys must match those ids.
- NSE continues to use `event_id` + stored session to build content (unchanged).

### android

- Same product: gateway base `https://push.flim.me` + Flim `app_id` + FCM via Sygnal `gcm`. Exact settings field names follow Element X Android when that tree is in play.
- Application id root remains **`me.flim`** (decision 000a).

### Gateway ops (`push-gateway/` in meta)

- Path: **`push-gateway/`** at the meta workspace root (alongside `ios/`, `android/`, `specs/`).
- Source: [element-hq/sygnal](https://github.com/element-hq/sygnal) (or its published container), pinned by the sub-project.
- Target: UpCloud Managed Kubernetes (Finland), **same namespace** as 008; public name **`push.flim.me`**.
- Sygnal `apps:` entries for each `app_id` the clients register — owned by this tree, not by a human one-off edit on the server.

## Open issues

_(none)_

## History

- 2026-10-14: first write
- 2026-10-14: lock gateway host to `push.flim.me`
- 2026-10-14: deploy via meta `push-gateway/` sub-project (not human); apps point at the gateway in implementation
- 2026-10-14: TLS via deploy scripts; DNS remains human; `AGENTS.md` Layout includes `push-gateway/`
