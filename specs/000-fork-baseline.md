---
id: 000
slug: fork-baseline
title: Fork baseline
status: done
created: 2026-09-09
amends: []
platforms: []
---

# 000 — Fork baseline

## Intent

This workspace is a fork of Element X (iOS first; Android and Web later). The product baseline is upstream Element X. Fork behaviour and product design are only what later decisions (and their `.design.md` files) in this directory record.

## Acceptance

- [x] Ordered decisions under `specs/` are the record of fork changes; a decision’s `.design.md` is replay-canonical product design for that id.
- [x] App trees keep upstream structure and follow their own `AGENTS.md`; they are git submodules of this meta repo, not a source dump.
- [x] No requirement to specify the whole messenger.

## Non-goals

Documenting or reimplementing Element X as a whole. Replacing upstream test suites.

## Notes

iOS already implements Matrix voice messages. Specs after this one describe *changes* to that (or unrelated fork features), not the existence of voice bubbles.

Non-trivial decisions get a companion `NNN-slug.design.md` (replay-canonical with the decision). This baseline has none.

## Verification

Meta only — no app tests.

## History

- 2026-09-09: proposed and adopted
