---
decision: 002
slug: flim-strings
created: 2026-09-11
---

# 002 — Local Flim string catalog (en + fi) (design)

Companion to `002-flim-strings.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Upstream Element X shares translations across clients via Localazy. Contributors must not edit generated `Localizable` files; new Element-bound English goes through a staging table until Element staff import it. This fork cannot (and should not) put fork-only product copy into that shared project: downloads would clobber local edits, and Element translators are not owners of our deltas.

Later fork UI (e.g. decision **001**) will need fork-owned labels and accessibility strings. This decision locks **where** that copy lives and which languages the catalog supports — and ships the empty catalog plumbing before any Flim keys exist.

## Product design

### Ownership

- **Two catalogs:** Element-owned strings stay on the upstream path. Fork-owned strings live in a separate **Flim** catalog that this workspace maintains.
- **Source of truth is the repo.** Authors edit the Flim catalog in git. No third-party translation service is required for Flim strings. A service may be adopted later without changing ownership of the catalog.
- **Element download stays Element-only.** Refreshing upstream translations must never delete, merge into, or regenerate the Flim catalog from Localazy.
- **Empty at birth is fine.** This decision is satisfied by a wired, empty Flim catalog. Keys are added when a later decision needs them.

### Locales

- **Supported locales:** English and Finnish. English authoring uses the same `en` / en-GB role as upstream Element (not a separate `en-US` Flim catalog).
- **Fallback:** if the user’s preferred language has no Flim translation, show English. Never show a bare key or an empty string when English exists.
- **Other languages:** out of scope until a later change expands the locale set.

### What goes where

- **Flim catalog:** copy that exists only because of a fork decision (new chrome, fork-specific accessibility wording, fork-only empty states, and the like).
- **Upstream catalog:** unchanged Element strings, including when fork UI can point at an existing Element phrase that already means the right thing (e.g. a screen title reused as a destination label).
- **Do not** use upstream’s “untranslated / pending Localazy” staging as the permanent home for Flim strings.
- **Do not** rewrite an Element key’s value in the Element catalog to mean something fork-specific; add a Flim key instead.

### Key discipline

- Prefer Element key-naming conventions so replay on Android/Web stays predictable (action / common / screen / a11y style prefixes).
- **No fork-only key prefix.** The separate Flim catalog is the namespace; a key may look like an Element key without colliding, because lookup always goes through the Flim API / table, never the Element one.
- When a Flim key is added, provide English and Finnish in the same change (or an immediate follow-up). Completing other decisions’ `done` is not blocked by an empty Flim catalog.

### Authoring workflow (product-level)

1. Wire the empty Flim catalog (this decision).
2. When a feature needs fork-only copy: add the English Flim string and its Finnish translation, then call through the Flim API. (add this to AGENTS.md)
3. Call sites that reuse Element wording keep using the Element API.

## Platform notes

- **iOS:** table/files named `Flim.strings` / `Flim.stringsdict`; generated accessor enum `FlimL10n`. Place under the existing Localizations tree (`en.lproj`, `fi.lproj`). Wire SwiftGen similarly to `Untranslated*`, but use the **locale-aware** lookup (same behaviour as `L10n`), not the Untranslated “always en” lookup. Do not put Flim keys in `Untranslated.strings` or `Localizable.strings`. Empty tables (or a single sentinel used only in unit tests) are acceptable until real keys exist.
- **Android / Web (replay):** same two-catalog rule, en (en-GB) + fi, English fallback; resource/module names may differ — keep “Flim” in the name so ownership stays obvious.

## Open issues

_(none)_

## History

- 2026-09-11: first write
- 2026-09-11: no fork key prefix — catalog separation is enough for namespacing
- 2026-09-11: infra-only until keys appear; English = en-GB; drop Finnish-vs-feature-`done` coupling
