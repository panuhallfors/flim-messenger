---
id: 011
slug: flim-public-site
title: Flim public site and client endpoints
status:
  ios: implementing
  android: deferred
created: 2026-10-14
amends: [000a]
platforms: [ios, android]
---

# 011 — Flim public site and client endpoints

## Intent

Flim’s public face and the links and telemetry the apps open are **Flim’s**, on **`https://flim.me`**, not Element’s. Users see Flim legal and help pages, Flim branding on OIDC consent, and (when they opt in) analytics and crash reports go to Flim’s own PostHog and Sentry — not Element’s. In-app log bug reports (rageshake) stay off for now.

Amends decision **000a**: the public site, AASA, OAuth redirect host, and associated-domains target for OIDC are no longer “human forking checklist only” — they are owned by this decision’s `website/` implementation and app wiring (team id and MapLibre keys remain human as in 000a).

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or opening the linked pages).

- [ ] Links from the app for the website, privacy, acceptable use, copyright, help topics, and analytics terms open **Flim** pages on `flim.me`, not `element.io`.
- [ ] On an OIDC / MAS consent screen for Flim, the client name, logo, terms, and privacy policy are Flim’s (on `flim.me`), not Element’s.
- [ ] OAuth redirect for the Flim app completes on **`flim.me`**, not `element.io`.
- [ ] When the user opts in to analytics, events go to **Flim’s** PostHog project (EU), not Element’s.
- [ ] Crash reports go to **Flim’s** Sentry project (EU), not Element’s; the app does **not** offer in-app log bug-report upload (rageshake disabled).

## Non-goals

- Running a Matrix homeserver, Element Web, or account-provisioning site on `flim.me`.
- Hosting the site on UpCloud or any self-managed web server — static hosting only ([statichost.eu](https://www.statichost.eu/)).
- Operating a [matrix-org/rageshake](https://github.com/matrix-org/rageshake) (or other) bug-report log server; leave that path disabled until a later decision.
- Replacing Sentry with a third-party “shake to Linear” product, or rewriting the upstream bug-report UI for Linear.
- App icon / launch artwork (decision 013).
- Writing final legal counsel-approved policy text as acceptance — policies ship as Flim-owned pages; counsel review is human out of band.
- Changing push gateway hosting (decision 010) or transcription hosting (decision 008).

## Notes

Upstream Element X hard-codes `element.io` for website, logo, legal, help, OAuth redirect, analytics terms, and Element Web / provisioning hosts; PostHog, Sentry, and rageshake come from secrets / app id (`ios` `AppSettings` and equivalent). Decision 000a left OIDC domain hosting as human forking setup; **this decision amends that** for the public site / AASA / OAuth host / associated domains, and owns the **client endpoint retarget** as fork product. 000a’s Team ID and MapLibre steps stay human.

**Human-owned:** DNS for `flim.me` → statichost.eu; PostHog Cloud EU and Sentry EU org/project creation and keys; Apple Team ID and MapLibre (000a). **Implementation:** meta-root **`website/`** (Eleventy → static deploy to statichost.eu) plus app settings / secrets / associated-domains wiring on each platform. Product shape is in the design.

Design (replay-canonical with this decision): [`011-flim-public-site.design.md`](011-flim-public-site.design.md)

## Verification

### ios

- Tests: **pending host** (`/verify-decision`) — `FlimPublicSiteAppSettingsTests`, `FlimElementCallAnalyticsTests` (decision 011); unit rung only
- Agent UI: none
- Meta: `website/` Eleventy tree + `statichost.yml` are part of this decision’s implementation (not an app submodule pin)

### android

- deferred (same product design; wire when Android ships)

### web

- out of scope as a Flim **product** platform; `website/` is marketing / legal / OIDC support only, not an Element X web client

## History

- 2026-10-14: proposed
- 2026-10-14: amends 000a (OIDC public site / AASA / OAuth host owned here); analytics URL `/privacy#analytics`; help stubs credit Element X
- 2026-10-14: note — decision 012 amends empty `accountProvisioningHost` → `mobile.flim.me`
- 2026-10-14: non-goal — app icon / launch artwork owned by decision 013
- 2026-10-18: implementing ios
- 2026-10-18: legal IA — app terms/privacy rewritten; Flim Summaries separate pages; Element Call analytics nil
