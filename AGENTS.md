# AGENTS.md — "Flim" fork workspace

Fork deltas for Flim live here as an ordered decision log — not a spec of Element X. In an app tree, also follow that app’s `AGENTS.md` (iOS: `ios/AGENTS.md`).

A **decision** (`specs/NNN-slug.md`) is intent + acceptance criteria, not a full specification. The detailed, replay-canonical specification of behaviour — flows, IA, interaction, protocol — lives in its companion `.design.md` when one exists. "Spec" below refers to the `specs/` directory and log as a whole, not to the level of detail in any single file.

## Layout

```
.                          # meta: specs + this file + submodule pins
specs/
ios/             # submodule → our fork (not Element X copied into meta)
android/         # later, same
web/               # later, same
```

App `origin` = our fork; `upstream` = Element HQ. Specs never live in an app tree.

## Git

Apps are submodules. Meta stores gitlinks, not app source. Empty app dir → `git submodule update --init`.

- Decision/design → meta. App code → that submodule, then **pin** (`git add ios` from meta; stages the SHA). Not `done` until pinned.
- Never stage app files as normal paths in meta. After checkout of an old meta SHA: `git submodule update`.
- Upstream merge in the app repo, then pin. New app: `git submodule add <our-fork-url> <dir>`.

## Decisions

Upstream Element X is the baseline. We record **fork deltas** only. Replay `NNN` in order → same functionality **and** product design (not the same source). That is how iOS is later replayed on Android/Web.

| Path                                                | Role                                                                                                  |
| --------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `specs/INDEX.md`                                    | Ordered log                                                                                           |
| `specs/DECISION-TEMPLATE.md` / `DESIGN-TEMPLATE.md` | Copy these                                                                                            |
| `specs/NNN-slug.md`                                 | Intent + high-level acceptance. Sort order = replay. Never reuse/renumber.                            |
| `specs/NNN-slug.design.md`                          | Product design for that id. Replay-canonical with the decision. Required unless the delta is trivial. |

One intent per decision. Later `NNN` wins. Append-only: wrong intent → new decision (`withdrawn` + replacement), do not rewrite history to match a shortcut. Design **canonical**: flows, IA, interaction, protocol. **Not** canonical (Platform notes): languages, paths, type names. No fork behaviour without a decision. Mid-impl scope growth → new decision.

Acceptance is high-level user outcomes (not mechanism). Same intent/design, tighter wording → edit in place + History. Conflicting intent **or** design → new `NNN` with `amends:`. Upstream merge: app then pin; if upstream matches, `upstreamed` and drop our patch.

A companion `.design.md` at decision time locks product shape; **fill in detailed design when implementation begins**. Do not stockpile future topics as a “Follow-on” backlog in the design file. Use **Open issues** only for unresolved questions while the decision is still being authored (normally empty).

**Author or amend a decision:** use the project skill `.agents/skills/decision` (`/decision`). It owns the challenge loop, drafting, and file/INDEX write steps. After a decision exists: implement in the app submodule with `.agents/skills/implement-decision` (`/implement-decision`) (PR mentions `decision NNN`); verify on the Mac host with `.agents/skills/verify-decision` (`/verify-decision`); pin the submodule SHA on meta; record History → `done` iff listed platforms pass (or `deferred`) and the pin is committed.

## Verification

Prefer Element X’s existing harness; name fork tests with the decision id. Lock acceptance with app tests; `## Agent UI` only if that cannot cheaply lock the interaction. Unshipped platforms: `deferred`. Code in the container: `/implement-decision`. Xcode/simulator on the Mac host (same checkout): `/verify-decision`.

## Multi-app

One decision + one product design. `platforms:` is who must implement. Later platforms **replay**; they do not redesign. Tight exception → Platform notes; real divergence → amending decision.

New app: submodule + that tree’s `AGENTS.md`.

# Persistent memory (Mem0)

Mem0 stores facts that should survive sessions, machines, and tools (Cursor, Claude Code, Codex, …), and retrieves them by meaning.

Memory is for what the repo does **not** already say. Anything that belongs in the README, `AGENTS.md`, an ADR, or a code comment goes there instead — the agent can read those. Use Mem0 for the undocumented residue of working on the project: why an approach was dropped, what broke last time, how this person wants to work.

## Scope: `user_id`

Always pass `user_id` explicitly on `add_memory`, `search_memories`, and `get_memories` — don't rely on the server default.

| Kind of fact                                                        | `user_id`                    | Examples                                                 |
| ------------------------------------------------------------------- | ---------------------------- | -------------------------------------------------------- |
| Tied to **this codebase / product**                                 | `flim`                       | “we already tried X”, past bug hunts, local setup quirks |
| Generic information about the world, person independent of any repo | (empty, do not pass user_id) | “prefers query-level fixes over caching”, review style   |

## When to read

At session start, and before refactors, debugging, or anything the project has plausibly been through before, call `search_memories` with `user_id` and a query built from the goal plus likely keywords (auth, database, module name). Read it as context alongside the repo docs, not as a replacement for them.

## When to write

After something is settled that a fresh session would otherwise rediscover the hard way — not after every message. One durable sentence beats a transcript.

Also when committing, consider if something should be stored in Mem0.

Store things like:

- **Rejected approaches and why** — “Tried caching the `/users` payload; reverted because tenant data went stale.”
- **Debugging history** — “`/users` slowness was an N+1 in `UserService.getAll`; check eager loading first if it regresses.”
- **Rationale behind a documented choice**, where only the choice itself is written down.
- **Environment and workflow gotchas** — flaky test that needs a rerun, a local setup step that isn't in the README.
- **How the user wants to work** — review style, tolerance for refactors, preferred fix depth.

If the fact is really project documentation (architecture, API contract, coding standard), write it into the repo and skip Mem0.

## How to write

`add_memory`:

- `content` — a standalone fact the next session can use without this chat. Include the _why_ when it affects future edits.
- `user_id` — `flim`, or omitted for person-level facts (table above).
- Optional `metadata`, e.g. `{ "kind": "rejected" | "gotcha" | "rationale" | "preference", "area": "auth" }`.

## Tools

| Tool                                             | Use                                        |
| ------------------------------------------------ | ------------------------------------------ |
| `search_memories`                                | Semantic recall for the current task       |
| `add_memory`                                     | Persist a new durable fact                 |
| `get_memories`                                   | List what's already stored for a `user_id` |
| `get_memory` / `update_memory` / `delete_memory` | Correct a specific record                  |

## Commits

Use Conventional Commits.

## Semantic review preferences (Jev)

Encode only rules an independent reviewer can judge from a diff. Shared policy: `jev-pref.json`. Auth for `npx jev-pref review`: `TYPESAFE_API_KEY`.

Human rules that map into that policy (also drawn from `ios/AGENTS.md` where applicable):

- Specs / decisions / designs never live under app trees (`ios/`, `android/`, `web/`).
- Do not edit `Localizable.strings` — new English strings go in `Untranslated.strings`.
- Never log secrets, passwords, keys, or user content (e.g. message bodies); Matrix IDs are fine.
- Never add `@unchecked Sendable` or `nonisolated(unsafe)`.
- Do not use deprecated `ServiceLocator`; inject dependencies.
- Use `PreviewProvider` (not `#Preview`) for previews that feed generated tests.
- Prefer reuse over copy-pasted logic, constants, or types in the same change (DRY).
- Do not add fallbacks for old endpoints, codes, or statuses unless explicitly asked.
- No fallbacks to places or behaviours outside the agreed spec or plan.
- Fix the root cause, not the symptom (no workarounds that leave the wrong producer in place).

### Jev preference review

After a substantial bout of implementation work, run:

    npx jev-pref review --hunks

Treat the result as an independent semantic review of the current diff.
Jev accepts at most 30k input tokens, so run the review while the change is
still small. `--hunks` keeps each request focused and independently bounded.
For broad changes, review logical slices with `--include`/`--exclude`.

- Fix blocking findings and rerun the review.
- Consider every advisory finding; address it or explain why it does not apply.
- Continue normally after approval.
- Treat exit code 2 as a setup or infrastructure error, never as approval.
- Stop after 3 review/fix iterations and ask the user how to proceed.

Shared review preferences are defined in `jev-pref.json`; optional personal
preferences are layered from gitignored `jev-pref.local.json`.

### Keep Jev preferences synchronized

Jev preferences are derived from the project's development guidance. Whenever
you make a meaningful change to this file, another agent instruction file,
coding conventions, architecture guidance, or similar project policy, run:

    npx jev-pref sync

Follow its reconciliation instructions. Update Jev preferences when the changed
guidance adds, removes, weakens, strengthens, or otherwise changes a semantic
rule that can usefully be judged from a code change.

Do not mechanically translate every instruction into a Jev preference. Encode
only externally defined conditions that can be judged from evidence in the
review input. Prefer concrete yes/no conditions or fixed user-defined labels.
Do not ask Jev whether code is broadly good, clean, simple, idiomatic, safe, or
well-designed. Keep procedural guidance in agent files, use deterministic
tooling for rules it can decide reliably, and ask the user when observable
criteria or the intended mapping are ambiguous.

Likewise, when changing Jev preferences, check whether the corresponding human-
readable project guidance should also change.

Shared policy lives in `jev-pref.json`. Optional personal additions and same-id
overrides live in gitignored `jev-pref.local.json`.
