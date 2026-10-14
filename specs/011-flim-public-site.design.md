---
decision: 011
slug: flim-public-site
created: 2026-10-14
---

# 011 — Flim public site and client endpoints (design)

Companion to `011-flim-public-site.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Upstream Element X points website, logo, legal, help, OAuth, and analytics-terms URLs at `element.io`, and uses Element’s PostHog, Sentry, and rageshake via secrets / `bugReportApplicationID`. After decision 000a, Flim is a separate install identity; consent and settings still show Element’s logo, terms, and backends unless retargeted. This decision **amends 000a** so the OIDC public site / AASA / OAuth host are product-owned here (not only a human forking note). Decision 010 already uses `push.flim.me` on UpCloud; the **apex** public site is separate static hosting, not that cluster.

Logo source for the site and OIDC: `specs/flim-logo-master*.jpeg` (light and dark versions) (exported into the website build as needed).

## Product design

### Public host

- Canonical site: **`https://flim.me`**.
- Hosting: **[statichost.eu](https://www.statichost.eu/)** (European static hosting). No origin web server on UpCloud for this site.
- DNS for `flim.me` (and `www` if used) is **human**; TLS is whatever statichost.eu provisions for the custom domain.
- Subdomains already owned by other decisions stay where they are (e.g. `push.flim.me` → 010).

### Site information architecture

Minimum pages / assets the apps and OIDC need (paths may be refined at implementation, but the **roles** stay):

| Role                                                                                   | Path (locked intent)                                                                                                          |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Marketing / product home                                                               | `/`                                                                                                                           |
| Privacy policy                                                                         | `/privacy`                                                                                                                    |
| Acceptable use / terms                                                                 | `/acceptable-use`                                                                                                             |
| Copyright                                                                              | `/copyright`                                                                                                                  |
| Analytics / cookie terms                                                               | **`/privacy#analytics`** (stable fragment; app `analyticsTermsURL`)                                                           |
| Help (encryption, device verification, chat backup, identity pinning, history sharing) | `/help` with anchors covering the same topics upstream links today — **short stubs** that link out to the corresponding Element X / Element help docs, and clearly state that **Flim is a modified version of Element X** |
| App logo for OIDC / web views                                                          | Stable image URL under `flim.me` (e.g. `/mobile-icon.png`), derived from the master logo                                      |
| OAuth redirect base                                                                    | `/oauth/ios/<bundle id>` (and later Android path pattern as Element X uses)                                                   |
| OIDC `clientURI` path                                                                  | `/apps/ios` (and later `/apps/android`) appended to `https://flim.me`                                                         |
| Apple Associated Domains                                                               | `/.well-known/apple-app-site-association` with `webcredentials` for the 000a id family in use (shipping `me.flim`, or interim `me.flim.dev` until cutover — plus relevant extensions) |

Landing content stays minimal (brand, short product line, links to store / legal). Policies are **Flim-authored**, covering Flim as controller for site + app telemetry, and pointing at Matrix homeserver / Flim transcription / push as separate processing where relevant. Exact counsel wording is filled when implementing / reviewing — not a product redesign.

Help pages are not a full rewrite of Element’s docs: each topic is a short Flim stub (credit line + link to the upstream Element X / element.io help page for detail).

### Meta sub-project (`website/`)

Site build and deploy are **implementation of this decision in the meta repo**, not an app submodule.

- Tree: **`website/`** at the meta root (sibling to `ios/`, `android/`, `push-gateway/`). Listed in `AGENTS.md` Layout.
- Generator: **[Eleventy](https://www.11ty.dev/)** (static output only).
- Deploy scripts / config target **statichost.eu** (git webhook or CLI as their docs require). Secrets for deploy stay out of git.
- Does not register DNS; assumes `flim.me` already points at statichost.

### App endpoint map

| Concern                                           | Product choice                                                                                                                                      |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Website, logo, copyright, acceptable use, privacy | `https://flim.me` (+ paths above)                                                                                                                   |
| Help topics                                       | Flim `/help` (or fragment URLs) on `flim.me`                                                                                                        |
| Analytics terms URL                               | **`https://flim.me/privacy#analytics`**                                                                                                             |
| OAuth redirect                                    | `https://flim.me/oauth/ios/<bundle id>` (Android when that client ships)                                                                            |
| OIDC client / logo / tos / policy URIs            | Same host as above (MAS requires one domain)                                                                                                        |
| PostHog                                           | **Enabled** — Flim project on **PostHog Cloud EU**; host + API key via secrets (not Element’s)                                                      |
| Sentry (app + Rust SDK DSNs)                      | **Enabled** — Flim org/project with **EU data storage**; DSNs via secrets                                                                           |
| Bug-report application id                         | Flim-owned id (e.g. `flim-ios` / platform variants), not `element-x-ios`                                                                            |
| Rageshake / in-app log upload                     | **Disabled** — no URL configured (empty / unset), so the upstream “report a problem” upload path stays off                                          |
| `accountProvisioningHost`                         | Unset in **011** — **amended by decision 012** to `mobile.flim.me` (do not keep Element’s)                                                          |
| `elementWebHosts`                                 | **Keep** Element’s hosts (`app.element.io`, staging, develop) so foreign Element Web permalinks still open in-app; they are not Flim product URLs  |

Consent: analytics / Sentry prompts remain opt-in as upstream; copy and “learn more” must describe **Flim**, not Element.

### Human setup

1. **DNS** — point `flim.me` (and optional `www`) at statichost.eu as their docs require.
2. **statichost.eu** — create the site / link the `website/` deploy path; hand any deploy tokens out of band.
3. **PostHog Cloud EU** — create Flim project; hand host + project API key into app secrets.
4. **Sentry EU** — create org with EU residency at signup; hand DSNs into app secrets; configure projects for iOS (and later Android / Rust as needed).
5. **Associated domains** — once AASA is live on `flim.me`, ensure the app entitlement uses `flim.me` (000a / FORKING.md); re-run project generation as that platform requires.
6. **Legal review** — human counsel pass on privacy / terms before store submission if required.

### Out of band

Push (`push.flim.me`), transcription service, MapLibre keys, and Apple team id remain under their existing decisions / 000a human checklist.

## Platform notes

### ios

- Retarget the website / legal / help / analytics-terms / OAuth / logo knobs in runtime settings (upstream: `AppSettings`).
- PostHog / Sentry via existing secrets mechanism (`POSTHOG_*`, `SENTRY_*`); leave rageshake secret unset so configuration is disabled.
- `bugReportApplicationID` → Flim id, not `element-x-ios`.
- Associated domain entitlement: `webcredentials:flim.me` (drop `*.element.io` for production Flim).
- Exact type / file names follow Element X iOS.

### android

- Same URL map and telemetry choices when the Android tree is in play; package id family follows 000a (shipping `me.flim`, interim `me.flim.dev` until cutover).
- Rageshake / equivalent log-upload endpoint unset.

### Site ops (`website/` in meta)

- Path: **`website/`** at meta root.
- Stack: Eleventy → static files → **statichost.eu**.
- Public name: **`https://flim.me`**.

## Open issues

_(none)_

## History

- 2026-10-14: first write
- 2026-10-14: lock analytics URL to `/privacy#analytics`; help = Element X stubs + credit; amends 000a OIDC site ownership
- 2026-10-14: lock keep `elementWebHosts`; provisioning host empty in 011 (filled by 012)
- 2026-10-14: AASA / package family — allow interim `me.flim.dev` per 000a until shipping cutover
