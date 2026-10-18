# Flim (fork meta)

Meta workspace for the **Flim** fork of Element X: ordered decisions under `specs/`, plus git submodule pins for the app forks (`ios/`, later `android/`). First-party trees (`push-gateway/`, `website/`) live here too. Product platforms are **iOS and Android** only. See [`AGENTS.md`](AGENTS.md) for layout, git rules, and the decision log.

**Authorship:** new Flim-authored code is copyright **Panu Hällfors** (not Element Creations / New Vector). Upstream Element headers stay on Element-origin files. Details in [`AGENTS.md`](AGENTS.md).

## Branch policy

Same shape on **meta and every subproject**: `develop` = work + test, `main` = published. Solo trunk — decisions in `specs/` are the product audit trail, not feature branches.

| Repo | `develop` | `main` | Tags |
|------|-----------|--------|------|
| **Meta** (this repo) | Specs, pins, and first-party trees under test. Day-to-day commits. Pins track each app’s `develop`. | Published product line (specs + pins + first-party that match what you ship). Advance only when cutting a release. | Optional release tags on ship. |
| **App** (`ios/`, later `android/`) | Only working branch. TestFlight / internal builds. | Store / published line. Advance only when shipping. | `release/…` for immutable store cuts. |
| **First-party** in meta (`push-gateway/`, `website/`) | Follow meta: change them on meta `develop`; ship with meta `main`. | — | — |

Rules:

- **No feature branches by default.** Short-lived branches only for throwaway experiments.
- **Day-to-day work → `develop`.** Decisions, pins, app code, gateway, site — all land on the relevant `develop`.
- **Upstream merges into app `develop` only** — never into `main`. Use [`/merge-upstream`](.agents/skills/merge-upstream/). Keep Element’s `develop` name so syncs stay `merge upstream/develop`.
- **Publish = promote `develop` → `main`** (merge or fast-forward) on the repos you are shipping; tag the shipped commit when you care about reproducibility.
- Hotfix published `main` only when you must patch without taking all of `develop`; otherwise fix on `develop` and cut again.

## Agent skills

Project skills live under [`.agents/skills/`](.agents/skills/) (invoke with `/name`):

| Skill | Use for |
|-------|---------|
| [`decision`](.agents/skills/decision/) | Author or amend a fork decision (+ design) in `specs/` |
| [`implement-decision`](.agents/skills/implement-decision/) | Implement a decision in the app submodule (devcontainer) |
| [`verify-decision`](.agents/skills/verify-decision/) | Xcode/simulator verify on the Mac host; mark that platform `done` when pinned |
| [`merge-upstream`](.agents/skills/merge-upstream/) | Merge Element HQ into the app fork, then pin the submodule on meta |
| [`review-for-bugs`](.agents/skills/review-for-bugs/) | Review session work for bugs, corner cases, AGENTS alignment, and DRY; fix without over-engineering |

**Upstream sync:** run [`/merge-upstream`](.agents/skills/merge-upstream/) — merge in the app repo (`origin` = our fork, `upstream` = Element HQ), land on app `develop`, then pin on **meta `develop`**. Do not merge Element X into this meta tree.
