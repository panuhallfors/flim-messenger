---
id: 013
slug: flim-app-icon
title: Flim app icon and launch artwork
status:
  ios: proposed
  android: proposed
created: 2026-10-14
amends: []
platforms: [ios, android]
---

# 013 — Flim app icon and launch artwork

## Intent

Everywhere the installed app (and its store listing) shows a product mark — home-screen icon, launch / splash, notification small icon, and store artwork — that mark is **Flim’s**, not Element’s. Masters live in the meta `specs/` tree; platforms ship correctly sized and formatted derivatives.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or looking at the store listing).

- [ ] On the home screen and in system app lists, the app icon is the Flim mark, not Element’s.
- [ ] While the app is launching (splash / launch screen), the artwork is Flim’s, not Element’s.
- [ ] Notification chrome that shows an app mark uses Flim’s mark, not Element’s.
- [ ] App Store / Play listing icons and promotional graphics for this product use the Flim mark, not Element’s.
- [ ] Light, dark, and (where the OS offers it) tinted / monochrome icon presentations all read as Flim, not as a leftover Element asset.

## Non-goals

- Redesigning the in-app visual language (decision 003) or the public-site / OIDC logo URL wiring (decision 011) — those may reuse the same masters, but this decision owns **installable-app and store** surfaces only.
- Replacing Element trademarks inside third-party or system chrome we do not control.
- Authoring marketing copy, screenshots, or store listing text (beyond the visual mark assets) — **decision 014**.
- Committing every platform size/format into `specs/` — masters stay in meta; sized exports land in the app / store tooling at implementation.

## Notes

000a and 003 explicitly left icons / launch artwork out of scope; 011 left them as a separate branding decision. **This is that decision.**

**Masters (source of truth):**

- Icons: `specs/flim-logo-master-light.jpeg`, `specs/flim-logo-master-dark.jpeg`, `specs/flim-logo-master-tinted.jpeg`
- Transparent (alpha) cut-out: `specs/flim-logo-master-transparent.png`
- Splash / launch: `specs/flim-splash-master.jpeg`

Implementation derives platform icon sets, notification glyphs, store upload sizes, and launch-screen crops from these (scaling, cropping, alpha, and format conversion). Do not invent a second logo in the apps.

Design (replay-canonical with this decision): [`013-flim-app-icon.design.md`](013-flim-app-icon.design.md)

## Verification

### ios

- Tests: none yet (asset / asset-catalog presence and launch configuration; confirm on device / simulator that Element assets are gone)
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-14: proposed
- 2026-10-14: splash master `specs/flim-splash-master.jpeg`; transparent logo masters tracked as open work
- 2026-10-18: transparent master `specs/flim-logo-master-transparent.png` landed; open work closed
- 2026-10-18: note — store listing copy / screenshots owned by decision 014
