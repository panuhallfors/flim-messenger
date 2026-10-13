# Decision index

Replay order = `id` ascending. Each id is the decision **plus** its `.design.md` when present. Later ids win on conflict. See [DECISION-TEMPLATE.md](DECISION-TEMPLATE.md), [DESIGN-TEMPLATE.md](DESIGN-TEMPLATE.md), top-level `AGENTS.md`, and `/decision` (`.agents/skills/decision`) to author a new entry.

| id | slug | status | intent | platforms | design |
|----|------|--------|--------|-----------|--------|
| 000 | [fork-baseline](000-fork-baseline.md) | done | This workspace records fork deltas on Element X; no full-app decision | meta | — |
| 000a | [flim-app-identity](000a-flim-app-identity.md) | proposed | Distinct Flim Messenger install identity (short: Flim) + our bundle/app-group ids; OIDC/maps human setup | ios | [design](000a-flim-app-identity.design.md) |
| 001 | [thumb-reach-room-exit](001-thumb-reach-room-exit.md) | proposed | Narrow-layout thumb-reachable room/space→list exit via container transform + bottom strip; wide/split stays upstream | ios | [design](001-thumb-reach-room-exit.design.md) |
| 002 | [flim-strings](002-flim-strings.md) | proposed | Local Flim string catalog infra (en-GB + fi); Element Localazy unchanged | ios | [design](002-flim-strings.design.md) |
| 003 | [flim-visual-refresh](003-flim-visual-refresh.md) | proposed | App-wide Flim visual restyle (neutral surfaces, glass controls, rainbow line/outline) on narrow and wide; no navigation change beyond 001; drops no upstream info; amends 001's visual detail | ios | [design](003-flim-visual-refresh.design.md) |
| 004 | [voice-message-brief](004-voice-message-brief.md) | proposed | Voice message shows title, summary, and transcript; other clients see title and summary only; mock service until a later decision | ios | [design](004-voice-message-brief.design.md) |
