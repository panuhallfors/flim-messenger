# Decision index

Replay order = `id` ascending. Each id is the decision **plus** its `.design.md` when present. Later ids win on conflict. See [DECISION-TEMPLATE.md](DECISION-TEMPLATE.md), [DESIGN-TEMPLATE.md](DESIGN-TEMPLATE.md), top-level `AGENTS.md`, and `/decision` (`.agents/skills/decision`) to author a new entry.

| id | slug | status | intent | platforms | design |
|----|------|--------|--------|-----------|--------|
| 000 | [fork-baseline](000-fork-baseline.md) | done | This workspace records fork deltas on Element X; no full-app decision | meta | — |
| 001 | [thumb-reach-room-exit](001-thumb-reach-room-exit.md) | proposed | Narrow-layout thumb-reachable room/space→list exit via container transform + bottom strip; wide/split stays upstream | ios | [design](001-thumb-reach-room-exit.design.md) |
| 002 | [flim-strings](002-flim-strings.md) | proposed | Local Flim string catalog infra (en-GB + fi); Element Localazy unchanged | ios | [design](002-flim-strings.design.md) |
