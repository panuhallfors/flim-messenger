# AGENTS.md — fork workspace

Fork deltas live here as an ordered spec log — not a spec of Element X. In an app tree, also follow that app’s `AGENTS.md` (iOS: `element-x-ios/AGENTS.md`).

## Layout

```
.                          # meta: specs + this file + submodule pins
specs/
element-x-ios/             # submodule → our fork (not Element X copied into meta)
element-x-android/         # later, same
element-web/               # later, same
```

App `origin` = our fork; `upstream` = Element HQ. Specs never live in an app tree.

## Git

Apps are submodules. Meta stores gitlinks, not app source. Empty app dir → `git submodule update --init`.

- Spec/design → meta. App code → that submodule, then **pin** (`git add element-x-ios` from meta; stages the SHA). Not `done` until pinned.
- Never stage app files as normal paths in meta. After checkout of an old meta SHA: `git submodule update`.
- Upstream merge in the app repo, then pin. New app: `git submodule add <our-fork-url> <dir>`.

## Specs

Upstream Element X is the baseline. We record **fork deltas** only. Replay `NNN` in order → same functionality **and** product design (not the same source). That is how iOS is later replayed on Android/Web.

| Path | Role |
|------|------|
| `specs/INDEX.md` | Ordered log |
| `specs/TEMPLATE.md` / `DESIGN-TEMPLATE.md` | Copy these |
| `specs/NNN-slug.md` | Intent + acceptance. Sort order = replay. Never reuse/renumber. |
| `specs/NNN-slug.design.md` | Product design for that id. Replay-canonical with the spec. Required unless the delta is trivial. |

One intent per spec. Later `NNN` wins. Append-only: wrong intent → new spec (`withdrawn` + replacement), do not rewrite history to match a shortcut.

Design **canonical**: flows, IA, interaction, protocol, rejected alternatives. **Not** canonical (Platform notes): languages, paths, type names.

No fork behaviour without a spec. Mid- impl scope growth → new spec.

### Workflow

1. Next `NNN` from `TEMPLATE.md`; row in `INDEX.md`; `proposed`.
2. Non-trivial: `NNN-slug.design.md`; link from INDEX + spec Notes.
3. Implement in the app submodule; PR mentions `spec NNN`.
4. Pin submodule SHA on meta; record in spec History.
5. Verify. `done` iff listed platforms pass (or `deferred`) and pin is committed.

Same intent/design, tighter wording → edit in place + History. Conflicting intent **or** design → new `NNN` with `amends:`.

Upstream merge: app then pin. If upstream matches, `upstreamed` and drop our patch.

## Verification

Use Element X’s existing tests first (unit, snapshots, UI, a11y). Stock voice-message tests are baseline, not fork specs. Do not re-test all of Element X. Name/comment fork tests with the spec id.

1. Acceptance bullets (observable, not implementation).
2. App tests (VM/service; snapshots; existing UITests).
3. `## Agent UI` in the spec — only when (1)–(2) cannot cheaply lock the interaction. No extra test framework.

Unshipped platforms: `deferred`.

## Multi-app

One spec + one product design. `platforms:` is who must implement. Later platforms **replay**; they do not redesign. Tight exception → Platform notes; real divergence → amending spec.

New app: submodule + that tree’s `AGENTS.md`.
