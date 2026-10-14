# Flim (fork meta)

Meta workspace for the **Flim** fork of Element X: ordered decisions under `specs/`, plus git submodule pins for the app forks (`ios/`, later `android/`). Product platforms are **iOS and Android** only. See [`AGENTS.md`](AGENTS.md) for layout, git rules, and the decision log.

## Agent skills

Project skills live under [`.agents/skills/`](.agents/skills/) (invoke with `/name`):

| Skill | Use for |
|-------|---------|
| [`decision`](.agents/skills/decision/) | Author or amend a fork decision (+ design) in `specs/` |
| [`implement-decision`](.agents/skills/implement-decision/) | Implement a decision in the app submodule (devcontainer) |
| [`verify-decision`](.agents/skills/verify-decision/) | Xcode/simulator verify on the Mac host; mark `done` when pinned |
| [`merge-upstream`](.agents/skills/merge-upstream/) | Merge Element HQ into the app fork, then pin the submodule on meta |

**Upstream sync:** run [`/merge-upstream`](.agents/skills/merge-upstream/) — merge in the app repo (`origin` = our fork, `upstream` = Element HQ), resolve conflicts against `specs/`, then pin with `git add ios` (or `android`) on meta. Do not merge Element X into this meta tree.
