---
decision: 014
slug: flim-store-listing
created: 2026-10-18
---

# 014 — Flim store listing (design)

Companion to `014-flim-store-listing.md`. **Replay-canonical** with that decision: later platforms implement this store presence, not a parallel one.

## Context

Decision **000a** owns the installed product’s app name (**Flim**) and package / bundle identity. Store presence was split out because it is console metadata and creative, not an `app.yml` knob. Decision **013** owns the Flim mark masters and store icon / feature-graphic derivatives. This decision owns the rest of a shippable store product page: name, copy fields, screenshots, and related console metadata.

## Product design

### Naming

- **App name** (store listing title / “name” field): **Flim** — same brief name as the home-screen app (000a).
- **Flim Messenger:** use only in **legal text and other long contexts** on the listing (and anywhere else product copy needs a longer formal name). Do not use it as the store app name or as a second short brand.

### Listing package (must be in place to ship)

A complete Flim listing includes, on each platform’s console:

| Surface | Requirement |
|--------|-------------|
| App name | **Flim** |
| Subtitle / short description | Present; Flim-branded; not Element |
| Description (and promo / “what’s new” style fields the store requires) | Present; Flim-branded; not Element |
| Screenshots (phone; tablet / other form factors if the store requires them for the binary) | Show Flim UI / branding, not Element X |
| Preview video | Optional; if used, same branding rule as screenshots |
| Keywords / tags / category / similar discoverability fields | Do not present the product as Element |
| Privacy / age / data-safety style console questionnaires | Filled for Flim; not left as Element leftovers |
| Legal / long-form naming inside listing copy | **Flim Messenger** when a longer name is needed |

Exact sentence-level marketing copy and screenshot shot lists are chosen at implementation / store-prep time; they must satisfy the table above.

### Boundary with 013

- Icon and feature graphic (and other *mark* uploads) come from **013**.
- Screenshots and listing *text* are this decision. Do not re-author a second logo inside screenshots; show the shipped Flim chrome.

### Ownership

- Humans (or store-prep tooling) set and maintain App Store Connect / Play Console fields and upload screenshots for the shipping apps. Acceptance is the live or submitted listing matching this design — not an in-app binary change alone.

## Platform notes

### ios

- App Store Connect: name, subtitle, promotional text, description, keywords, screenshots / preview, category, age rating, privacy nutrition labels, etc.
- App name field = **Flim**. Not `APP_DISPLAY_NAME` in `app.yml` (that remains 000a’s install-time display name, also **Flim**).

### android

- Play Console: title, short/full description, screenshots / feature graphic (mark from 013), store listing graphic set, category, content rating, Data safety, etc.
- Title = **Flim**. Launcher label stays **Flim** per 000a.
- Platform status for this decision is **deferred** until the Android app ships.

## Open issues

_(none)_

## History

- 2026-10-18: first write — title split from 000a
- 2026-10-18: full listing package; app name **Flim**; **Flim Messenger** for legal/long contexts only
