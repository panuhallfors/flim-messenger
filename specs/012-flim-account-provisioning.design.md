---
decision: 012
slug: flim-account-provisioning
created: 2026-10-14
---

# 012 — Flim account provisioning links (design)

Companion to `012-flim-account-provisioning.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Element documents mobile provisioning links on `https://mobile.element.io/…?account_provider=…&login_hint=…`. Element X already turns matching Universal Links into an auth start aimed at that homeserver. Decision 011 cleared Flim’s provisioning host so Element’s site would not be used. This decision fills that gap with a Flim host for easier onboarding (email link or QR), with **`matrix.org`** as Flim’s default account provider for that flow (matching upstream’s default managed provider).

## Product design

### Host and URL shape

- Public host: **`mobile.flim.me`** (not the apex marketing site alone — keeps provisioning Universal Links off unrelated `flim.me` paths).
- Same static hosting as decision 011 ([statichost.eu](https://www.statichost.eu/) via `website/`). DNS for the subdomain is human.
- Link shape (aligned with Element’s documented query names):

  `https://mobile.flim.me/?account_provider=<serverName>&login_hint=<optional>`

  Examples:

  - `https://mobile.flim.me/?account_provider=matrix.org` (canonical default)
  - `https://mobile.flim.me/?account_provider=example.com`
  - `https://mobile.flim.me/?account_provider=example.com&login_hint=mxid:@alice:example.com`

- **Default homeserver:** **`matrix.org`**. If `account_provider` is omitted, the site and (where the app receives such a URL) the app treat the provider as `matrix.org`. Prefer publishing links that **include** `account_provider=matrix.org` explicitly for clarity.
- Path may be `/` or a short path such as `/flim` for clarity; the app matches on **host** + query items (upstream behaviour). Prefer one canonical path in implementation and document it on the site.

### QR helper (required)

- `mobile.flim.me` includes a **QR page** (same host; path chosen at implementation) that:
  - Defaults `account_provider` to **`matrix.org`**.
  - Lets the user optionally set another server name and an optional login hint.
  - Shows a scannable QR of the resulting HTTPS provisioning URL, and the URL as text (copyable).
  - States the link is for **Flim** (modified Element X where helpful), not Element.
- No server-side account storage — pure client-side / static generation of the URL and QR image.

### App behaviour

- `accountProvisioningHost` = **`mobile.flim.me`**.
- With Flim installed and associated domains live, opening the link starts the existing provisioning / auth flow for that `account_provider` and optional `login_hint`.
- Missing `account_provider` → use **`matrix.org`** (same default as upstream’s first managed account provider).
- Do not use `mobile.element.io`.

### Site behaviour (no app / fallback)

- Opening a provisioning URL when the app is not installed shows a static page that:
  - Recognises the same query parameters (for display — “this link is for server X”, defaulting display to `matrix.org` when omitted).
  - States that the link is for **Flim**, and that Flim is a modified version of Element X where helpful.
  - Offers a clear path to install Flim (store badges / links when available; placeholder copy until stores exist).
  - Does not silently redirect to Element’s app or `mobile.element.io`.

### Associated domains

- AASA on `mobile.flim.me` includes **`applinks`** for the Flim app id family (decision 000a), in addition to whatever 011 requires on `flim.me` for `webcredentials`.
- App entitlement includes `applinks:mobile.flim.me`.
- Android: equivalent App Links for the same host when that platform ships.

### Human setup

1. DNS — point `mobile.flim.me` at the same static host as `flim.me`.
2. Confirm Universal Links / App Links after deploy (open a sample link / scan the default QR on a device with Flim installed).

### Out of band

MAS/IdP login-hint forwarding on a given HS, and MDM, remain outside this decision. Flim does not operate `matrix.org`.

## Platform notes

### ios

- Set provisioning host to `mobile.flim.me`; add `applinks:mobile.flim.me` beside 011’s `flim.me` webcredentials / other applinks kept from upstream (e.g. `matrix.to`, Element Web hosts as in 011).
- Existing `AccountProvisioningURLParser` / auth start path — no new auth UI; ensure omitted provider falls back to `matrix.org` if upstream requires an explicit value.
- Keep upstream’s default account provider list headed by `matrix.org` unless a later decision changes it.

### android

- Same host, default, and query shape; App Links + intent filters when Android is implemented.

### Site (`website/`)

- Fallback page + **QR helper** under the 011 Eleventy tree; deploy still statichost.eu.

## Open issues

_(none)_

## History

- 2026-10-14: first write
- 2026-10-14: default `matrix.org`; QR helper required; host `mobile.flim.me`
