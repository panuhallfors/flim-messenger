---
decision: 013
slug: flim-app-icon
created: 2026-10-14
---

# 013 — Flim app icon and launch artwork (design)

Companion to `013-flim-app-icon.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Upstream Element X ships Element-branded app icons, launch screens, notification marks, and store-facing artwork. After decision 000a, Flim is a separate install identity; after 003 it has its own in-app look — but home-screen and launch surfaces still show Element’s mark unless replaced. Decision 011 already points the **web / OIDC** logo at the same master files under `specs/`; this decision owns the **native install and store** surfaces that 000a / 003 / 011 deferred.

## Product design

### Mark and masters

- The product mark is the headset + chat-bubble glass mark in the masters — not Element’s logo, not a wordmark-only glyph.
- Canonical masters in meta (do not duplicate a divergent design in an app tree):
  - **Light** — `specs/flim-logo-master-light.jpeg` (full-colour mark on a light field).
  - **Dark** — `specs/flim-logo-master-dark.jpeg` (full-colour mark on a dark field).
  - **Tinted** — `specs/flim-logo-master-tinted.jpeg` (monochrome / single-hue glass mark for OS tinted or equivalent presentations).
  - **Splash** — `specs/flim-splash-master.jpeg` (full-bleed launch composition: mark on the dark radial field).
- Implementation **scales and reformats** these masters into each platform’s required sizes and file types (PNG/WebP/asset catalogs, adaptive layers, etc.). Exact pixel grids and toolchains are filled when implementation begins; the product rule is: every shipped surface is derived from these masters (or a clear, documented simplification of the same mark for very small sizes if the full glass detail does not hold).
- Current logo masters are JPEG with baked fields. **Transparent (alpha) versions** of light / dark / tinted are required before adaptive foregrounds, notification glyphs, and other surfaces that need a cut-out mark — tracked under Open issues until those files land in `specs/`.

### Where the mark appears

| Surface | Product rule |
| ------- | ------------ |
| Home-screen / launcher icon | Flim mark; OS light / dark / tinted (or Android adaptive) variants as the platform supports |
| Launch / splash screen | Composition from `flim-splash-master.jpeg` (crop / letterbox per device as needed); no Element artwork |
| Notification app icon / small icon | Flim mark (simplified if the platform requires a flat / monochrome small glyph — still recognisably the same mark) |
| Store listing icon and required brand graphics | Flim mark from the logo masters |

### Appearance variants

- **Light** and **dark** follow the light and dark masters so the icon stays readable on both system appearances.
- **Tinted / monochrome** (e.g. iOS tinted icon mode) uses the tinted master — not a washed Element asset and not an ad-hoc recolour of the full rainbow mark unless the OS forces a system tint on a provided template.
- **Android adaptive icon:** foreground is the Flim mark with alpha (with safe-zone padding); background is a solid field consistent with the light or dark master field, not Element’s adaptive layers. Prefer the transparent logo masters once they exist.

### Relationship to other decisions

- **003** restyles in-app chrome; it does not replace the OS icon or launch image.
- **011** may export a web/OIDC logo from the same masters; that URL is not a substitute for native icon assets.
- In-app chrome that shows a product logo (if any beyond launch) should use the same mark family — but inventing new in-app logo placements is out of scope unless they already exist upstream as Element artwork to replace.

## Platform notes

- **iOS:** Replace Element asset-catalog app icons (including dark and tinted appearances where the catalog supports them), launch screen / splash assets, notification-related app marks, and App Store icon upload sources. Paths and catalog names follow upstream’s forking layout once implementation starts.
- **Android:** Replace launcher / adaptive icon resources, splash / launch theme drawables, notification small icon, and Play Console icon / feature-graphic sources. Adaptive foreground/background layering as above.

## Open issues

- Produce **transparent (alpha) versions** of the three logo masters (light, dark, tinted) and land them in `specs/` (e.g. PNG), suitable for adaptive icon foregrounds, notification glyphs, and other cut-out uses. JPEG masters with baked fields stay as the full-bleed references until then.

## History

- 2026-10-14: first write
- 2026-10-14: splash master `specs/flim-splash-master.jpeg`; open issue for transparent logo masters
