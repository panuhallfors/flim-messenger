---
name: decision
description: >-
  Author a new fork decision and optional companion design in specs/. Use when
  the user runs /decision, asks to create or amend a decision, write a
  NNN-slug.md / .design.md, or add a fork delta to the decision log.
disable-model-invocation: true
---

# Decision authoring

Create (or amend via a new id) one fork **decision** and, when non-trivial, its companion **design**. Do not implement app code in this skill.

Templates: `specs/DECISION-TEMPLATE.md`, `specs/DESIGN-TEMPLATE.md`. Index: `specs/INDEX.md`. Orientation: top-level `AGENTS.md`.

## What these files are

| File | Role |
|------|------|
| `specs/NNN-slug.md` | Intent + high-level acceptance. Sort order = replay. Never reuse/renumber. |
| `specs/NNN-slug.design.md` | Product design for that id. Replay-canonical with the decision. Required unless the delta is trivial. |
| `specs/INDEX.md` | Ordered log row for every decision |

Upstream Element X is the baseline. Record **fork deltas** only. Replay `NNN` in order → same functionality **and** product design (not the same source).

- One intent per decision. Later `NNN` wins.
- Append-only: wrong intent → new decision (`withdrawn` + replacement); do not rewrite history to match a shortcut.
- Design **canonical**: flows, IA, interaction, protocol, rejected alternatives.
- **Not** canonical (Platform notes): languages, paths, type names.
- No fork behaviour without a decision. Mid-impl scope growth → new decision.
- Same intent/design, tighter wording → edit in place + History entry.
- Conflicting intent **or** design → new `NNN` with `amends:`.
- Upstream merge that matches ours → `upstreamed` and drop our patch (outside this skill’s write path; still know the rule).

## Acceptance is high level

A decision states **outcomes for the user**, not mechanism. A handful of bullets, each confirmable by using the app, each still true under any reasonable implementation of the design.

Mechanism, thresholds, timings, glyphs, copy, layout, control names, and edge-case handling belong in `.design.md`. If a bullet would change when the design changes, it is too specific.

Design detail growing is a design edit (History entry), not a new decision. Only a changed _intent or product design_ needs a new `NNN`.

## Workflow (do not skip the challenge loop)

Prefer files over chat for the draft body. Write the markdown early; iterate by editing those files. Use chat for questions, challenges, and short summaries of what changed — not for pasting the full decision/design.

Copy this checklist and track it:

```
Decision progress:
- [ ] 1. Discover — read context; enough clarifying Qs to pick shape
- [ ] 2. Write drafts — files + INDEX; status proposed
- [ ] 3. Challenge & edit — push back; revise the .md files until the user is happy
- [ ] 4. Confirm — summarize paths and open follow-ons
```

### 1. Discover

Read `specs/INDEX.md` and any related existing decisions/designs. Search Mem0 (`user_id: flim`) for prior rejects and preferences on this area.

Ask only what you need to choose `NNN`, slug, platforms, design yes/no, and a coherent first draft. Cover gaps among:

- **User outcome** — what changes versus upstream (or versus an amended decision)?
- **Scope** — one intent or several? Split if several.
- **Non-goals** — what must stay out so scope cannot silently grow?
- **Platforms** — who must implement now (`ios` / `android` / `web`)? Others stay `deferred`.
- **Design needed?** — new flow, protocol, cross-app shape, or non-obvious upstream reuse → yes. One-line behaviour tweak → maybe none.
- **Amends** — does this conflict with an earlier decision/design? If yes, plan `amends:` and possibly `withdrawn` on the old id.
- **Verification sketch** — will acceptance be lockable with existing app tests, or might `## Agent UI` be needed later?

Lock shape for the first write:

1. Next `NNN` = highest id in `INDEX.md` + 1 (zero-pad to 3 digits).
2. `slug` (kebab-case), `title`, `platforms`, `amends`.
3. Whether a `.design.md` will exist.
4. Status: `proposed`.

If a long thread already produced several candidates, write up the **last one the user settled on**, not a synthesis of all of them — unless they ask to merge.

Do not wait for a perfect plan before writing files. Get enough to draft; refine on disk.

### 2. Write drafts (early)

As soon as shape is clear, create the files — do not hold the draft in chat:

1. Copy templates → `specs/NNN-slug.md` and, if needed, `specs/NNN-slug.design.md`.
2. Fill frontmatter and a complete best-effort body (intent, acceptance, non-goals, notes; design sections if present).
3. Link the design from the decision Notes and from `INDEX.md`.
4. Append a row to `INDEX.md` (`proposed`, intent one-liner, platforms, design link or `—`).
5. Decision History: `YYYY-MM-DD: proposed`. Design History: `YYYY-MM-DD: first write`.

Acceptance: handful of user-observable bullets. Mechanism that creeps in → move into the design file.

Design: Context, Product design, Alternatives rejected, Platform notes (translation only), Follow-on, History. Rejected alternatives stay rejected in replay unless a later decision reopens them.

Point the user at the file paths so they can read the draft as markdown. Do not commit unless the user asks.

### 3. Challenge and edit files

Do not rubber-stamp. Until the user says the decision is good, challenge and **apply agreed changes by editing the files** (and INDEX intent one-liner if it drifts). Add a History line when intent/design wording meaningfully changes under the same id.

Challenge checklist:

- Is this actually a fork delta, or already true upstream?
- Is the intent one decision or a bundle that should be split?
- Are acceptance bullets outcome-level, or sneaking in mechanism?
- Are non-goals strong enough to block the usual scope creep?
- For design: which alternatives were considered, and why rejected? Missing rejects are a smell.
- Multi-app: is the product design replayable, or are we baking in one platform’s toolkit?
- Does an existing decision already cover this (edit + History) vs need a new `NNN`?

Present challenges as concrete objections or alternatives. After each round, edit the `.md` files and briefly say what changed (paths + short delta), not a full re-paste.

If slug or “needs design” changes mid-flight, rename/add/remove files and fix INDEX + Notes links in the same turn.

Repeat until the user is happy with the realized decision (and design, if any).

### 4. Confirm

Report final paths, id/slug, whether design exists, and any Follow-on items left for later decisions.

## After authoring (not this skill’s job, but know the handoff)

1. Implement in the app submodule; PR mentions `decision NNN`.
2. Pin submodule SHA on meta; record in decision History.
3. Verify per `AGENTS.md` (acceptance → app tests → Agent UI only if needed). `done` iff listed platforms pass (or `deferred`) and pin is committed.

## Multi-app reminder

One decision + one product design. `platforms:` is who must implement. Later platforms **replay**; they do not redesign. Tight exception → Platform notes; real divergence → amending decision.
