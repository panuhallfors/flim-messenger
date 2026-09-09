---
id: NNN
slug: short-kebab-name
title: Human title
status: proposed   # proposed | implementing | done | withdrawn | upstreamed
created: YYYY-MM-DD
amends: []         # e.g. [001]
platforms: [ios]   # ios | android | web
---

# NNN — Title

## Intent

One or two sentences. What changes for the user versus upstream (or versus the amended spec)?

## Acceptance

Observable checks. Replay = these become true after this spec, given all earlier specs.

- [ ] …
- [ ] …

## Non-goals

What this spec does not do (avoids silent scope growth).

## Notes

Upstream hooks, MSCs, existing types to reuse, platform exceptions.

Design (non-trivial, replay-canonical with this spec): `NNN-slug.design.md` | none

## Verification

### ios

- Tests: (paths or “none yet”)
- Agent UI: none | see below

### android

- deferred

### web

- deferred

## Agent UI

Not a test framework. Optional script for an AI agent to drive the **real app** (simulator / emulator / browser) and report pass/fail against acceptance bullets.

**Default: omit the steps and set `Agent UI: none` above.** Prefer unit tests, snapshots, and the app’s existing UI tests.

Write steps only when the intent is interaction that those tests cannot cheaply lock (hold-to-record, mic permission, audio actually playing, etc.). Each step must map to an acceptance bullet. Keep it short and deterministic.

1. …  # maps to acceptance: “…”
2. …

## History

- YYYY-MM-DD: proposed
