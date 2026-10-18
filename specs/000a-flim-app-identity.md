---
id: 000a
slug: flim-app-identity
title: Flim app identity (display name + bundle IDs)
status:
  ios: implementing
  android: deferred
created: 2026-09-11
amends: []
platforms: [ios, android]
---

# 000a — Flim app identity (display name + bundle IDs)

## Intent

The installed app is a distinct product identity from Element X on every platform: it shows as **Flim** on the home screen and in system lists (store listing / long-form name **Flim Messenger**), and it uses our own package / bundle / app-group identifiers so it can sit alongside Element X without colliding.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or looking at the installed app on device).

- [ ] On the home screen and in system app lists, the app’s display name is **Flim**, not Element X.
- [ ] In the App Store / Play store listing, the product name is **Flim Messenger**.
- [ ] Where the product uses a short name in UI copy, that name is **Flim**, not Element.
- [ ] UI that names **this** product (notification sound labels, forced-upgrade copy, and similar) says **Flim**, not Element; other products named Element Call / Pro / Classic may stay.
- [ ] Installing this fork does not replace or share the same app identity as Element X — both can be installed at once as separate apps.
- [ ] App extensions that ship with the fork (notifications, share, etc.) belong to the same Flim identity family, not Element’s.

## Non-goals

- Agent implementation of OIDC / MAS public site, associated domains, MapLibre keys, or other runtime fork setup — those were **human tasks** in this decision’s design; **decision 011 amends** ownership of the public site / AASA / OAuth host / associated-domains target (Team ID and MapLibre stay human here).
- Changing icons, accent colour, or other brand chrome beyond display name + identifiers.
- Offering migration from Element Classic (the old Element / Vector iOS app) or from Element X — do not claim Element’s classic app-group / keychain entitlements under our team.
- Choosing or documenting a Nightly / alternate build flavour.

## Notes

Upstream documents the minimum project knobs in `ios/docs/FORKING.md` (`app.yml` → `xcodegen`). This decision records the **fork identity values** plus the human setup checklist for the rest of that forking guide.

**Interim bundle ids:** Shipping identity remains **`me.flim`**. Until the final Apple (and Play) developer account owns those App IDs, builds use **`me.flim.dev`** / **`group.me.flim.dev`** so a temporary team cannot claim the shipping id. Design spells the cutover.

**Classic migration (context):** Element X can optionally read accounts/session material from the pre-X “Element Classic” iOS app via shared app group + keychain (`CLASSIC_*` in `app.yml`). That path is Element-owned; Flim does not implement or retarget it.

Design (replay-canonical with this decision): [`000a-flim-app-identity.design.md`](000a-flim-app-identity.design.md)

## Verification

### ios

- Tests: **pending host** (`/verify-decision`) — `FlimAppIdentityTests` (unit); manual: home-screen **Flim**, notification sound labels, bundle id family `me.flim.dev*`; store listing **Flim Messenger** is App Store Connect (human). Side-by-side with Element X when both installed.
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-09-11: proposed
- 2026-09-11: full/short names (Flim Messenger / Flim); OIDC+rest of FORKING.md as human tasks; classic migration clarified as non-goal
- 2026-10-14: platforms = ios, android (same product identity on every platform)
- 2026-10-14: note — decision 011 amends OIDC public-site / AASA / OAuth host ownership
- 2026-10-14: home screen / system lists = Flim; store listing = Flim Messenger (avoids launcher truncation)
- 2026-10-14: acceptance — this-product “Element” UI copy → Flim (design: audit + Flim keys per 002)
- 2026-10-14: note — interim `me.flim.dev` until final developer account owns shipping `me.flim`
- 2026-10-14: implementing
- 2026-10-18: status per platform — ios implementing, android deferred
