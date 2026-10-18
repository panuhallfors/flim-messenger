---
id: 014
slug: flim-store-listing
title: Flim store listing
status:
  ios: proposed
  android: deferred
created: 2026-10-18
amends: [000a]
platforms: [ios, android]
---

# 014 — Flim store listing

## Intent

App Store and Play product pages present **Flim** as the app (not Element X): listing name, marketing fields, screenshots, and related console metadata are complete and Flim-branded before ship. Long-form **Flim Messenger** appears only in legal text and other long contexts — not as the store app name.

## Acceptance

**High level.** Outcomes a person could confirm by looking at the store listing (or the submitted store metadata).

- [ ] The store listing’s app name is **Flim**, not Element X / Element and not **Flim Messenger**.
- [ ] Required marketing fields for a shippable listing (subtitle / short description, full description, and any platform-required promo text) are filled and present Flim, not Element.
- [ ] Listing screenshots (and preview video if used) show the Flim app, not Element X.
- [ ] Keywords, category, and other discoverability fields do not brand the product as Element.
- [ ] Where the listing includes legal or other long-form product naming, that text uses **Flim Messenger** (not Element) when a longer name is needed.
- [ ] Store listing mark assets (icon / feature graphic) follow decision **013**; they are present for the listing this decision ships.

## Non-goals

- Home-screen / system-list display name, in-app short product name, or bundle / package / app-group identifiers — still decision **000a** (app name there is also **Flim**).
- Authoring the pixel masters for icons / splash / feature graphic — decision **013** (this decision consumes those store-side assets).
- Counsel-approved final legal policy pages — decision **011** / human counsel; this decision only requires the *listing* legal/long naming convention above.
- Changing in-app companion copy that already uses long-form **Flim Messenger** (e.g. decision 004).

## Notes

Amends **000a**, which previously bundled a longer store title with install identity. Store listing content lives in App Store Connect / Play Console (plus screenshot sources in store tooling), not in `app.yml` / Android application label knobs.

Naming: **Flim** = app name everywhere the product is named briefly (install + store). **Flim Messenger** = legal text and other long contexts only.

Design (replay-canonical with this decision): [`014-flim-store-listing.design.md`](014-flim-store-listing.design.md)

## Verification

### ios

- Tests: none in-app (console metadata + screenshot assets). Human check of App Store Connect / live listing when published.
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-18: proposed — split store listing out of 000a (title only)
- 2026-10-18: full listing package (marketing fields + screenshots); app name **Flim**; **Flim Messenger** for legal/long contexts; android deferred
