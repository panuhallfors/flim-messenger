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

The installed app is a distinct product identity from Element X on every platform: it shows as **Flim** on the home screen and in system lists, and it uses our own package / bundle / app-group identifiers so it can sit alongside Element X without colliding.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or looking at the installed app on device).

- [x] On the home screen and in system app lists, the app’s display name is **Flim**, not Element X. (`FlimAppIdentityTests.displayAndProductionNamesAreFlim` — `CFBundleDisplayName` / `bundleDisplayName`)
- [x] Where the product uses a short name in UI copy, that name is **Flim**, not Element. (`productionAppName` via same test)
- [ ] UI that names **this** product (notification sound labels, forced-upgrade copy, and similar) says **Flim**, not Element; other products named Element Call / Pro / Classic may stay. (Flim catalog / call sites — decision 002; not locked by `FlimAppIdentityTests`)
- [x] Installing this fork does not replace or share the same app identity as Element X — both can be installed at once as separate apps. (`interimBaseBundleIdentifier` — `me.flim.dev` / `group.me.flim.dev`)
- [x] App extensions that ship with the fork (notifications, share, etc.) belong to the same Flim identity family, not Element’s. (`FlimAppIdentityTests.embeddedExtensionsShareFlimIdentityFamily` — NSE / ShareExtension bundle ids, base, app group, display / production names)

## Non-goals

- App Store / Play **listing package** (name, copy, screenshots, console metadata) — **decision 014** (amends this decision for that ownership). Listing app name there is also **Flim**; **Flim Messenger** is for legal / long contexts only.
- Agent implementation of OIDC / MAS public site, associated domains, MapLibre keys, or other runtime fork setup — those were **human tasks** in this decision’s design; **decision 011 amends** ownership of the public site / AASA / OAuth host / associated-domains target (Team ID and MapLibre stay human here).
- Changing icons, accent colour, or other brand chrome beyond display name + identifiers (store artwork → **013**).
- Offering migration from Element Classic (the old Element / Vector iOS app) or from Element X — do not claim Element’s classic app-group / keychain entitlements under our team.
- Choosing or documenting a Nightly / alternate build flavour.

## Notes

Upstream documents the minimum project knobs in `ios/docs/FORKING.md` (`app.yml` → `xcodegen`). This decision records the **fork identity values** plus the human setup checklist for the rest of that forking guide.

**Naming:** App name is **Flim** (home screen, system lists, in-app short product label). **Flim Messenger** is reserved for legal text and other long contexts (see also **014** for store listing copy).

**Interim bundle ids:** Shipping identity remains `me.flim`. Until the final Apple (and Play) developer account owns those App IDs, builds use `me.flim.dev` / `group.me.flim.dev` so a temporary team cannot claim the shipping id. Design spells the cutover.

**Classic migration (context):** Element X can optionally read accounts/session material from the pre-X “Element Classic” iOS app via shared app group + keychain (`CLASSIC_*` in `app.yml`). That path is Element-owned; Flim does not implement or retarget it.

Design (replay-canonical with this decision): [`000a-flim-app-identity.design.md`](000a-flim-app-identity.design.md)

## Verification

### ios

- Tests (2026-10-18 host): `swift run tools ci run-tests --scheme UnitTests --test-name FlimAppIdentityTests` — all 4 passed (`displayAndProductionNamesAreFlim`, `interimBaseBundleIdentifier`, `backgroundTaskIdentifiersDeriveFromBaseBundleIdentifier`, `embeddedExtensionsShareFlimIdentityFamily`). Preview/UI rungs skipped (no snapshot surface).
- Still human / not locked by that suite: product-self “Element” copy (notification sounds etc. → 002); side-by-side install when Element X is also present.
- Agent UI: none
- Status: stay `implementing` until ios submodule SHA pinned on meta `develop`.

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
- 2026-10-18: ios host verify — `FlimAppIdentityTests` (UnitTests) passed; acceptance partial (product-self copy / extension asserts still open); pin → `done` when asked
- 2026-10-18: store listing ownership moved to decision 014 (amends); intent/acceptance narrowed to install identity
- 2026-10-18: naming — app name **Flim**; **Flim Messenger** only in legal/long contexts; extension identity machine-checked via `embeddedExtensionsShareFlimIdentityFamily`
- 2026-10-18: pinned ios @ 22626e025
