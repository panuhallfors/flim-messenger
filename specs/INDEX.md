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
| 005 | [voice-message-transcription](005-voice-message-transcription.md) | proposed | Settings choice of Flim, a custom server, or on-device transcription; the user finishes that method's setup before it is used; Flim is email, then a promo or a subscription | ios, android | [design](005-voice-message-transcription.design.md) |
| 006 | [custom-server-transcription](006-custom-server-transcription.md) | proposed | When custom server is in use, produce the brief via that OpenAI-compatible host with app-owned prompts; replaces the mock on that path only | ios, android | [design](006-custom-server-transcription.design.md) |
| 007 | [on-device-transcription](007-on-device-transcription.md) | proposed | When on-device is in use, produce the brief on the phone from the OS speech and language models; the phone must be able to transcribe and summarise locally, and the audio stays on the device | ios, android | [design](007-on-device-transcription.design.md) |
| 008 | [flim-hosted-transcription](008-flim-hosted-transcription.md) | proposed | When Flim is in use, produce the brief via Flim's EU-hosted service (Cortecs whisper + small LLM); replaces the mock on that path only | ios, android | [design](008-flim-hosted-transcription.design.md) |
| 009 | [transcription-setup-prompts](009-transcription-setup-prompts.md) | proposed | Soft post-login transcription setup ask; stronger CTA on first unset voice message; not-set-up opens setup not try again | ios, android | [design](009-transcription-setup-prompts.design.md) |
