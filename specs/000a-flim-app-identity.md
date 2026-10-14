---
id: 000a
slug: flim-app-identity
title: Flim app identity (display name + bundle IDs)
status: proposed
created: 2026-09-11
amends: []
platforms: [ios, android]
---

# 000a — Flim app identity (display name + bundle IDs)

## Intent

The installed app is a distinct product identity from Element X on every platform: it shows as **Flim Messenger** (short name **Flim** where the product uses a short label), and it uses our own package / bundle / app-group identifiers so it can sit alongside Element X without colliding.

## Acceptance

**High level.** Outcomes a person could confirm by using the app (or looking at the installed app on device).

- [ ] On the home screen and in system app lists, the app’s display name is **Flim Messenger**, not Element X.
- [ ] Where the product uses a short name in UI copy, that name is **Flim**, not Element.
- [ ] Installing this fork does not replace or share the same app identity as Element X — both can be installed at once as separate apps.
- [ ] App extensions that ship with the fork (notifications, share, etc.) belong to the same Flim identity family, not Element’s.

## Non-goals

- Agent implementation of OIDC / MAS, associated domains, MapLibre keys, or other runtime fork setup — those are **human tasks** listed in the design, not coding acceptance for this decision.
- Changing icons, accent colour, or other brand chrome beyond display name + identifiers.
- Offering migration from Element Classic (the old Element / Vector iOS app) or from Element X — do not claim Element’s classic app-group / keychain entitlements under our team.
- Choosing or documenting a Nightly / alternate build flavour.

## Notes

Upstream documents the minimum project knobs in `ios/docs/FORKING.md` (`app.yml` → `xcodegen`). This decision records the **fork identity values** plus the human setup checklist for the rest of that forking guide.

**Classic migration (context):** Element X can optionally read accounts/session material from the pre-X “Element Classic” iOS app via shared app group + keychain (`CLASSIC_*` in `app.yml`). That path is Element-owned; Flim does not implement or retarget it.

Design (replay-canonical with this decision): [`000a-flim-app-identity.design.md`](000a-flim-app-identity.design.md)

## Verification

### ios

- Tests: none yet (identity is largely build/config; confirm via installed app metadata / side-by-side install with Element X)
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-09-11: proposed
- 2026-09-11: full/short names (Flim Messenger / Flim); OIDC+rest of FORKING.md as human tasks; classic migration clarified as non-goal
- 2026-10-14: platforms = ios, android (same product identity on every platform)
