# Decision index

Replay order = `id` ascending. Each id is the decision **plus** its `.design.md` when present. Later ids win on conflict. See [DECISION-TEMPLATE.md](DECISION-TEMPLATE.md), [DESIGN-TEMPLATE.md](DESIGN-TEMPLATE.md), top-level `AGENTS.md`, and `/decision` (`.cursor/skills/decision`) to author a new entry.

| id | slug | status | intent | platforms | design |
|----|------|--------|--------|-----------|--------|
| 000 | [fork-baseline](000-fork-baseline.md) | done | This workspace records fork deltas on Element X; no full-app decision | meta | — |
| 001 | [thumb-reach-room-exit](001-thumb-reach-room-exit.md) | proposed | Thumb-reachable room→list exit via container transform + bottom strip | ios | [design](001-thumb-reach-room-exit.design.md) |
