---
name: verify-decision
description: >-
  Run scoped Xcode/simulator verification for a fork decision on the Mac host
  (only the rungs this decision needs), re-record snapshots when needed, then
  fill Verification and mark done when pinned. Use when the user runs
  /verify-decision, asks to verify decision NNN on the host, or continue
  after /implement-decision with simulator tests.
disable-model-invocation: true
---

# Verify a decision (Mac host)

Run the Xcode test ladder for an existing `specs/NNN-slug.md` after `/implement-decision` (or equivalent app changes). Do **not** author decisions or implement feature code here unless a failing test forces a tiny fix — then re-run the ladder.

Orientation: top-level `AGENTS.md`. App conventions: `ios/AGENTS.md`. Snapshots: `ios/.agents/skills/update-snapshots` (`/update-snapshots`).

## Environment (required)

**Must run on the Mac host with Xcode** (not the Linux devcontainer).

- Abort (tell the user to switch to a host Cursor/terminal session) if `xcodebuild` / usable Xcode is missing, or if you are clearly inside Linux without host tooling.

If the user names no id, ask which `NNN` (or confirm the obvious `implementing` row in `specs/INDEX.md`).

## Preconditions

- Decision status is usually `implementing`; app changes for this id exist in the submodule.
- Read the decision’s `## Verification` (may say pending host) and the implement plan’s Host verify subsection if available in chat/transcript.
- Search Mem0 (`user_id: flim`) for gotchas on this area / flaky tests.

## Checklist

Copy and track:

```
Verify progress:
- [ ] 1. Load — decision, Verification pending, platform AGENTS
- [ ] 2. Tests — only rungs this decision needs (scoped); name fork tests with NNN
- [ ] 3. Snapshots — /update-snapshots only if refs need refresh
- [ ] 4. Record — Verification + acceptance evidence; pin when asked → done
```

### 1. Load

1. `specs/NNN-slug.md` — Acceptance, Verification, `platforms:`
2. `specs/NNN-slug.design.md` if linked — only as needed to judge intentional UI diffs
3. Platform `AGENTS.md` for platforms under test
4. Confirm host Xcode is available

### 2. Tests (scoped; optional rungs)

Policy (`AGENTS.md` Verification): prefer Element X’s existing harness; do not re-test the whole app; name/comment fork tests with decision `NNN`. Cover this decision’s surface only.

Prefer Xcode MCP tools over raw terminal when available.

**Not every rung every time.** Skip a rung when that surface did not change. Prefer scoped `--test-name` (or the few relevant classes) — never the whole UnitTests / PreviewTests / UITests suite “just because.”

| Rung | Run when | iOS (from `ios/`) |
|------|----------|-------------------|
| Unit / VM | Logic, state, coordinators for this decision | `swift run tools ci run-tests --scheme UnitTests --test-name …` (full `unit-tests` only if many classes touched) |
| Preview (simulator) | `TestablePreview` / snapshot surface for this decision | Scoped PreviewTests; device iPhone SE (3rd gen), OS = `CI.defaultOSVersion`. Full `preview-tests` only if many previews touched |
| UI (simulator) | Only if acceptance needs a UITest flow / UITest snapshot Preview cannot lock | Scoped UITests (`ui-tests` + `--test-name` / matching classes). iPad only if the decision touches wide/split. **Skip entirely** for most decisions |

Stop and fix before climbing further. Verified = the **chosen** scoped runs pass — not that every scheme ran.

Fill `## Verification` → platform section with what ran (test names / schemes). Add `## Agent UI` only if those tests still cannot cheaply lock acceptance — no new test framework.

### 3. Snapshots (if needed)

If Preview/UI tests fail on image mismatch, or new screens lack refs:

1. Confirm the visual change is intentional and matches the design.
2. Run `/update-snapshots` (read and follow `ios/.agents/skills/update-snapshots/SKILL.md`).
3. Re-run the same simulator tests **without** record mode; they must pass.
4. Leave `RECORD_FAILURES` off in any `*.xctestplan`.

Skip when no snapshot surface changed. CI alternative: `record-snapshots` label on an ios PR (see that skill).

### 4. Record status (meta)

Update on meta (not `done` until pin is committed):

- Decision `## Verification` — tests that lock acceptance; Agent UI if any.
- Mark acceptance checkboxes only with evidence from tests (or Agent UI).
- Decision + INDEX `status`: stay `implementing` until listed platforms pass **and** the submodule pin is committed on meta → then `done`.
- History: pin line when relevant (`YYYY-MM-DD: pinned ios @ <sha>`).
- Pin: from meta, `git add ios` (gitlink only) when the user asks to commit/pin.

Do not push or open a PR unless asked; if a PR is opened in the app fork, title/body mention `decision NNN`.

## Done means

`done` iff every platform in `platforms:` is verified (or explicitly `deferred`) **and** the app submodule SHA is pinned on meta. Until then: `implementing`.

## Out of scope for this skill

- Authoring / amending decision intent → `/decision`
- Full feature implementation / Plan-mode coding → `/implement-decision`
- Writing implementation detail into `.design.md`
- Upstream merge / `upstreamed` status
