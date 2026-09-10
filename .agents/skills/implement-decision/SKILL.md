---
name: implement-decision
description: >-
  Implement a fork decision in the listed app submodule(s): plan in Plan mode
  first, code only after the user accepts the plan. Stops before Xcode/simulator
  tests — those run separately on the Mac host via /verify-decision. Use when
  the user runs /implement-decision, asks to implement decision NNN, or build a
  specs/ decision end-to-end (coding phase).
disable-model-invocation: true
---

# Implement a decision

Turn an existing `specs/NNN-slug.md` (+ `.design.md` when present) into working app code. Do **not** author a new decision here — that is `/decision`. Do **not** run Xcode / simulator tests here — that is `/verify-decision` on the Mac host.

Orientation: top-level `AGENTS.md`. App conventions: that platform’s `AGENTS.md` (iOS: `ios/AGENTS.md`).

## Environment

This skill is meant for the **devcontainer (Linux)**. The workspace is typically the same bind-mounted checkout the Mac host uses — no push/pull required before host verify.

Never invoke `xcodebuild`, `swift run tools ci …`, simulators, or `/update-snapshots` from this skill. If the session is already on macOS with Xcode, still finish implement here, then run `/verify-decision` as a separate step (do not collapse them unless the user explicitly asks).

## Preconditions

- Decision files exist and the user has settled intent/design (status usually `proposed`).
- App submodule present (`git submodule update --init` if empty).
- Search Mem0 (`user_id: flim`) for prior rejects/gotchas on this area before planning.

If the user names no id, ask which `NNN` (or read `specs/INDEX.md` and confirm the obvious one).

## Checklist

Copy and track:

```
Implement progress:
- [ ] 1. Load — decision, design, platform AGENTS, Mem0
- [ ] 2. Scope lock — platforms now; non-goals; mid-impl growth → /decision
- [ ] 3. Plan — SwitchMode plan; detailed impl plan + host-verify handoff; wait for user accept
- [ ] 4. Implement — app submodule only; follow the accepted plan
- [ ] 5. Handoff — Verification pending host; tell user to run /verify-decision
```

### 1. Load

Read, in order:

1. `specs/NNN-slug.md` — Intent, Acceptance, Non-goals, Verification, `platforms:`
2. `specs/NNN-slug.design.md` if linked — product shape (canonical for *what* the product does)
3. `specs/INDEX.md` row for this id
4. Platform `AGENTS.md` for each platform in `platforms:` that is not deferred
5. Mem0 search for the feature area

Set status to `implementing` in the decision frontmatter **and** INDEX when work starts. History: `YYYY-MM-DD: implementing`.

### 2. Scope lock

- Implement **only** listed platforms; others stay `deferred` in Verification.
- Non-goals are hard stops. Scope growth mid-impl → stop and run `/decision` (new id); do not stretch this one.
- Acceptance stays high-level user outcomes. Do not rewrite acceptance into mechanism while implementing.

### 3. Plan (required before any coding)

**Hard gate:** no app-code edits or test scaffolding until the user accepts the plan.

1. Call `SwitchMode` with `target_mode_id: plan` (explain: need an accepted implementation plan before coding).
2. In Plan mode, produce a **detailed implementation plan**. Put mechanism and engineering detail here — files/types to touch, coordinators/VMs/views, navigation wiring, data flow, edge-case handling, which fork tests to add (named with decision `NNN`), and expected snapshot impact.
3. Always include a **Host verify** subsection: which `/verify-decision` rungs to run on the Mac (often unit only; Preview and/or UI only if this decision’s surface needs them), with scoped `--test-name`s / classes — not full suites by default. Same checkout; separate step.
4. Source of truth for product shape remains the decision + `.design.md`. The plan translates that into *how we will build it* on this platform.
5. **Do not** push plan detail into `.design.md` (or the decision). No “design deepening” write-back of tokens, type names, paths, or impl steps. If the plan reveals a product/intent conflict, stop and amend via `/decision` — do not paper over it in the design file.
6. Iterate the plan with the user until they explicitly accept it.
7. Only then leave Plan mode / proceed to Implement.

### 4. Implement

Work **inside the app submodule** (e.g. `ios/`). Never stage app source as normal paths on meta.

- Execute the **accepted plan**; if reality forces a material deviation, re-enter Plan mode and get acceptance again before continuing.
- Follow that app’s architecture and style (`ios/AGENTS.md`: MVVM-C, Compound, Swift concurrency, previews with `TestablePreview`, etc.).
- Add or update fork tests/comments that mention `decision NNN` (PRs too, when opened) — but do not execute them here.
- Do not commit unless the user asks. Conventional Commits in the submodule.
- Still **do not** dump impl detail into `.design.md`.

### 5. Handoff (end of this skill)

Update meta lightly — stay `implementing`; do **not** mark acceptance checkboxes or claim `done`:

- Decision `## Verification` → platform section: note tests are **pending host** (`/verify-decision`), optionally listing the planned schemes/`--test-name`s from the accepted plan.
- Leave Agent UI alone unless the plan already required it (usually filled during verify).
- Tell the user clearly: run `/verify-decision` for decision `NNN` on the **Mac host**, in this same repo checkout (bind-mounted; no push/pull needed).

Do not push or open a PR unless asked; if a PR is opened in the app fork, title/body mention `decision NNN`.

## Done means (not this skill)

`done` is decided by `/verify-decision` (platforms verified or `deferred`) **and** a committed submodule pin on meta. After implement alone, status stays `implementing`.

## Out of scope for this skill

- Authoring / amending decision intent → `/decision`
- Xcode / simulator / snapshot runs → `/verify-decision`
- Writing implementation detail into `.design.md`
- Upstream merge / `upstreamed` status
- Implementing deferred platforms “while we’re here”
