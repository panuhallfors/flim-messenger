---
decision: 000a
slug: flim-app-identity
created: 2026-09-11
---

# 000a — Flim app identity (design)

Companion to `000a-flim-app-identity.md`. **Replay-canonical** with that decision: later platforms implement this product identity, not a parallel one.

## Context

Upstream Element X ships as “Element X” under Element’s Apple identifiers (`io.element.elementx`, `group.io.element`, Element’s team). Flim is a fork that must be a separate installable product. Upstream’s forking notes (`ios/docs/FORKING.md`) call out display name, base bundle identifier, app group, and development team as the first project-configuration step, then OIDC/associated domains and MapLibre secrets as further setup.

## Product design

### Display names

- **Home screen / system app lists:** **Flim** (short enough to avoid launcher truncation; “Flim Messenger” is ~14 characters and ellipsizes on iOS and many Android launchers).
- **Store listing title** (App Store / Play): **Flim Messenger**.
- **Short name** (in-app product label where the stack interpolates a short production name): **Flim**.

Long-form **Flim Messenger** in other product copy (notices, legal, marketing sentences) is unchanged unless a later decision says otherwise.

**Element string audit:** Upstream’s English catalog has a handful of strings that say “Element” literally. Most name other products (Element Call, Element Pro, Element Classic) and stay. Strings that present **this** app as Element — e.g. notification sound names “Element Default” / “Element Fade”, and “use the new Element X app” — must resolve to **Flim** via Flim catalog keys at those call sites. Decision 002 forbids patching Element keys in place.

### Bundle / package identity

- Canonical reverse-DNS product id: **`me.flim`**.
- Rationale vs `…film-ios`: product spelling is **Flim**; platform suffix is unnecessary — each platform uses its own packaging field, and iOS extensions already hang off the base id with their own suffixes.
- App group id for shared container / capabilities: **`group.me.flim`** (same suffix as the base id, `group.` prefix).
- Extension and test targets derive from the base id (same pattern as upstream: e.g. notification service, share extension, unit/UI test bundles). Exact suffix strings are platform translation.

### Side-by-side with Element X

- Flim’s identifiers must not reuse Element’s base bundle id or app group. Users may keep both apps installed; data stores and push registrations stay separate.

### Signing team

- Builds sign with **our** Apple Development Team (not Element’s). The concrete Team ID is filled by a human (see below); it is not a user-facing product string.

### Human setup (from upstream forking guide)

These complete a usable fork after identity config. They are **human-owned**; they are not agent acceptance for this decision and do not need a separate decision id unless product behaviour diverges from upstream’s forking recipe.

1. Set Apple **Development Team ID** and regenerate the Xcode project (`xcodegen`).
2. **OIDC / MAS (amended by decision 011)** — public host `flim.me`, AASA, OAuth redirect paths, and associated-domains entitlement are owned by **011** (`website/` + app wiring), not as a free-form human checklist under this decision. Humans still do DNS and any Apple portal steps 011 lists.
3. **MapLibre / location** — obtain an API key, keep it out of git (`Secrets` / env / `pkl` as upstream suggests), configure map style IDs in runtime settings (upstream defaults such as `basic-v2` / `basic-v2-dark` are fine).
4. Any further `AppSettings` runtime tweaks beyond identity — human judgment unless a later decision owns them (011 owns legal / help / telemetry endpoints).

### Classic migration

Out of product scope. Do not enable or retarget Element Classic account import under Flim’s team/identifiers.

## Platform notes

### ios

- Identity knobs in `app.yml` (then `xcodegen`): `APP_DISPLAY_NAME` → **Flim**; `PRODUCTION_APP_NAME` → **Flim**; `BASE_BUNDLE_IDENTIFIER` → `me.flim`; `APP_GROUP_IDENTIFIER` → `group.me.flim`; `DEVELOPMENT_TEAM` → human-supplied Team ID.
- Store listing title **Flim Messenger** is App Store Connect metadata, not an `app.yml` knob.
- Do not retarget `CLASSIC_*` at Element’s groups; leave Classic migration disabled / unused for Flim.
- Nightly / flavour overlays (e.g. `Components/Variants/Nightly/`) are out of scope until a later decision.

### android

- Application id / package root: **`me.flim`**.
- Launcher / application label → **Flim**; Play listing title → **Flim Messenger**; short in-app product label → **Flim**.
- Do not reuse Element’s application id or shared storage; side-by-side install with Element X must work.

## Open issues

_(none)_

## History

- 2026-09-11: first write
- 2026-09-11: Flim Messenger (full) / Flim (short); human setup checklist for OIDC + MapLibre + team id; classic migration noted as out of scope
- 2026-10-14: android in platforms with ios (same names and reverse-DNS id); web out of product scope
- 2026-10-14: OIDC public site / AASA / OAuth host amended by decision 011
- 2026-10-14: home screen / launcher = Flim; store listing = Flim Messenger (avoids truncation)
- 2026-10-14: Element string audit — product-self hits → Flim keys at call sites (002); leave Call / Pro / Classic
