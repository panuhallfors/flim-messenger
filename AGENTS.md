# AGENTS.md — fork workspace

Fork deltas live here as an ordered decision log — not a spec of Element X. In an app tree, also follow that app’s `AGENTS.md` (iOS: `ios/AGENTS.md`).

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

One intent per decision. Later `NNN` wins. Append-only: wrong intent → new decision (`withdrawn` + replacement), do not rewrite history to match a shortcut. Design **canonical**: flows, IA, interaction, protocol, rejected alternatives. **Not** canonical (Platform notes): languages, paths, type names. No fork behaviour without a decision. Mid-impl scope growth → new decision.

Acceptance is high-level user outcomes (not mechanism). Same intent/design, tighter wording → edit in place + History. Conflicting intent **or** design → new `NNN` with `amends:`. Upstream merge: app then pin; if upstream matches, `upstreamed` and drop our patch.

**Author or amend a decision:** use the project skill `.cursor/skills/decision` (`/decision`). It owns the challenge loop, drafting, and file/INDEX write steps. After a decision exists: implement in the app submodule (PR mentions `decision NNN`); pin the submodule SHA on meta; record History; verify → `done` iff listed platforms pass (or `deferred`) and the pin is committed.

## Verification

Use Element X’s existing tests first (unit, snapshots, UI, a11y). Stock voice-message tests are baseline, not fork decisions. Do not re-test all of Element X. Name/comment fork tests with the decision id.

1. Acceptance bullets (user-observable outcomes; the design says how). Tests may assert design detail — acceptance itself stays high level.
2. App tests (VM/service; snapshots; existing UITests).
3. `## Agent UI` in the decision — only when (1)–(2) cannot cheaply lock the interaction. No extra test framework.

Unshipped platforms: `deferred`.

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
