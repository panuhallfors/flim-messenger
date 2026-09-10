---
id: 002
slug: flim-strings
title: Local Flim string catalog (en + fi)
status: proposed
created: 2026-09-11
amends: []
platforms: [ios]
---

# 002 — Local Flim string catalog (en + fi)

## Intent

This fork gets its own string catalog for fork-only copy — maintained locally in the repo for **English (en-GB)** and **Finnish**, not in Element HQ’s Localazy project. Upstream Element strings stay on the existing Localazy / `Localizable` path. This decision ships the **empty infrastructure**; the first Flim keys arrive with later decisions that need them.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] Fork-only copy is resolved from the Flim catalog, not from Element’s shared translation files.
- [ ] That catalog supports English (en-GB) and Finnish; if the device language is neither, fork-only copy appears in English rather than as a missing or raw key.
- [ ] Refreshing Element’s shared translations does not remove or overwrite the Flim catalog.
- [ ] Fork UI reuses existing Element wording when the meaning is already covered upstream, instead of inventing a parallel phrase for the same idea.

## Non-goals

- Adding fork keys to Element HQ Localazy, or requiring the Localazy CLI to author or translate fork copy.
- Localizing every Element string into Finnish ourselves — only fork-owned strings are in scope.
- Shipping Flim keys in this decision — the catalog may be empty until a later decision needs copy.
- Shipping additional locales beyond English (en-GB) and Finnish (more languages need a later decision or an in-place History expansion of locales).
- A separate American-English (`en-US`) Flim catalog.
- Permanently parking fork copy in upstream’s `Untranslated` staging table.
- Changing the meaning of an existing Element string key in place; fork chrome uses the Flim catalog (or an unchanged upstream key), not a patched `Localizable` value.

## Notes

Upstream (iOS): Element strings are downloaded into `Localizable.*` via Localazy; new upstream-bound English staging uses `Untranslated.*` → `UntranslatedL10n`. This decision does not replace that pipeline for Element-owned strings. Upstream’s `en` is en-GB; Flim matches that.

Design (replay-canonical with this decision): [`002-flim-strings.design.md`](002-flim-strings.design.md)

## Verification

### ios

- Tests: none yet (expect a small localization unit test once the catalog is wired)
- Agent UI: none

### android

- deferred

### web

- deferred

## History

- 2026-09-11: proposed
- 2026-09-11: infra-only (no keys yet); English clarified as en-GB; no en-US Flim locale
