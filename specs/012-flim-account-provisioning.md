---
id: 012
slug: flim-account-provisioning
title: Flim account provisioning links
status: proposed
created: 2026-10-14
amends: [011]
platforms: [ios, android]
---

# 012 — Flim account provisioning links

## Intent

Someone can send (or show as a **QR code**) a **Flim link** that opens the app already aimed at a Matrix homeserver, with an optional login hint — same ease as Element’s `mobile.element.io` links, without using Element’s host. Flim’s **default** homeserver for that flow is **`matrix.org`** (other servers still work when named in the link).

Amends decision **011**: Flim’s `accountProvisioningHost` is no longer empty; it is **`mobile.flim.me`**, served from the same `website/` / statichost.eu setup.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or scanning / opening the link).

- [ ] Opening a Flim provisioning link (or scanning its QR) on a device with Flim installed starts sign-in aimed at the homeserver named in the link (optional login hint applied when present).
- [ ] When the link does not name a homeserver, sign-in is aimed at **`matrix.org`**.
- [ ] The same link on a device **without** Flim opens a page on Flim’s site that explains the link and points the person to get Flim (not Element).
- [ ] Provisioning links use **`mobile.flim.me`**, not `mobile.element.io`.
- [ ] The site offers a **QR** for a provisioning link (defaulting to `matrix.org`, with optional other server / login hint).

## Non-goals

- Operating `matrix.org` or any homeserver — only recommending it as the default target in Flim’s provisioning UX.
- MDM / AppConfig delivery of account provider.
- Changing OIDC / MAS consent URLs, legal pages, PostHog, or Sentry (011).
- A full admin console or authenticated link builder — a public QR / link page is enough.
- Classic Element `hs_url` compatibility for the old Element app.

## Notes

Upstream already parses provisioning Universal Links when `accountProvisioningHost` matches the URL host and `account_provider` / `login_hint` query items are present. This decision retargets that host to Flim and adds the public page, QR helper, and associated-domain wiring on `website/` (decision 011’s tree). Upstream Element X already lists `matrix.org` as the default account provider in settings; this decision aligns provisioning defaults with that.

**Human-owned:** DNS for `mobile.flim.me` → same static host as `flim.me`. **Implementation:** extend `website/` + set the app’s provisioning host and `applinks` for that name.

Design (replay-canonical with this decision): [`012-flim-account-provisioning.design.md`](012-flim-account-provisioning.design.md)

## Verification

### ios

- Tests: none yet (host string and route parsing for a sample URL are lockable in unit tests; Universal Link end-to-end needs a device)
- Agent UI: none
- Meta: page + QR helper + AASA `applinks` live under `website/` (011)

### android

- deferred (same product design; App Links when Android ships)

### web

- out of scope as a chat client; `mobile.flim.me` is a static landing / Universal Link / QR target only

## History

- 2026-10-14: proposed
- 2026-10-14: default homeserver `matrix.org`; QR helper required; host stays `mobile.flim.me`
