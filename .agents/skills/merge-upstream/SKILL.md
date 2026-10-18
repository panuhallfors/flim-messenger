---
name: merge-upstream
description: >-
  Merge Element HQ Element X into a Flim app fork submodule, resolve conflicts
  against specs/, then pin the submodule SHA on meta. Use when the user runs
  /merge-upstream, asks to sync or merge upstream Element X, pull Element HQ
  into the fork, or update the ios/android submodule from upstream.
disable-model-invocation: true
---

# Merge upstream Element X

Bring Element HQ’s Element X into **our app fork** (submodule), re-apply or drop Flim deltas per `specs/`, then **pin** the resulting SHA on meta. Do **not** merge Element X into the meta repo. Do **not** invent new product behaviour during a sync — that is `/decision`.

Orientation: top-level `AGENTS.md`. App conventions: that platform’s `AGENTS.md` (iOS: `ios/AGENTS.md`).

## Topology (do not confuse these)

| Place | Role |
|-------|------|
| Meta (`/`, this repo) | Decisions/designs under `specs/`; **gitlink pins** only for app trees |
| App submodule (`ios/`, later `android/`) | Fork source. `origin` = our fork; `upstream` = Element HQ |
| Specs | Fork deltas only. Never live under an app tree |

Remotes (iOS today):

- `origin` → our fork (e.g. `…/flim-element-x-ios`)
- `upstream` → `element-hq/element-x-ios`

**Branch policy** (see top-level `README.md`): same on app and meta — `develop` = work + test, `main` = published.

| Place | Upstream sync does |
|-------|--------------------|
| App `develop` | Merge `upstream/develop` here. Never into app `main`. |
| Meta `develop` | Pin the landed app SHA here. Never pin a sync onto meta `main`. |

Confirm remotes with `git remote -v` inside the submodule before fetching. Keep the app name `develop` so syncs match Element HQ.

## Preconditions

- App submodule present (`git submodule update --init` if empty).
- Working tree in the submodule is clean enough to merge (stash or commit first).
- Search Mem0 (`user_id: flim`) for prior upstream-sync gotchas before resolving conflicts.
- Read `specs/INDEX.md` so conflict resolution and `upstreamed` checks have a checklist of live deltas.

If the user names no platform, default to `ios` when only that submodule exists; ask if both are present.

## Checklist

Copy and track:

```
Upstream merge progress:
- [ ] 1. Scope — platform submodule; confirm origin/upstream remotes
- [ ] 2. Fetch — upstream tip; update local develop from origin
- [ ] 3. Merge — prefer merge (not rebase) of upstream/develop into develop
- [ ] 4. Conflicts — keep upstream structure; re-apply Flim per specs/; upstreamed → drop patch
- [ ] 5. Prove — xcodegen/build/tests as appropriate; no new product inventing
- [ ] 6. Land & pin — push app origin develop; on meta develop, `git add <app>` pin (only if user asks to commit)
```

### 1. Scope

Work **inside the app submodule** (e.g. `cd ios`). Never stage app source as normal paths on meta.

One platform per run unless the user explicitly wants both. Android, when present, is the same loop against Element X Android upstream.

### 2. Fetch

```bash
cd ios   # or android/
git fetch upstream
git checkout develop
git pull origin develop
```

Optional: a short-lived `sync/upstream-YYYYMMDD` branch for review, then fast-forward / merge into `develop`. Default for this solo trunk: merge **on `develop`**. Do not track `upstream/develop` as the local branch tip. Do not merge into `main`.

### 3. Merge (not rebase)

```bash
git merge upstream/develop
```

**Prefer merge over rebase** once the fork has real Flim commits. Rebasing rewrites fork history and makes every later sync harder. Do not squash the upstream merge into a fake linear history.

### 4. Resolve conflicts against decisions

When conflicted:

1. Prefer **upstream** structure, APIs, and Element-owned behaviour unless a decision says otherwise.
2. Re-apply the **Flim delta** from `specs/NNN-slug.md` + companion `.design.md` (and the existing fork patch).
3. If upstream now matches a decision’s intent → **drop** our patch; in meta, set every listed platform’s status (and INDEX cells) to `upstreamed` and add a History line. That status change is part of this skill when the match is clear.
4. Identity / branding / strings / push / icons: follow the relevant decisions (e.g. 000a, 002, 010, 013) — do not “restore Element” in those surfaces.
5. Mid-sync product questions or new behaviour → stop and run `/decision`; do not stretch the merge PR.

Watch for coordinated bumps (e.g. `matrix-rust-components-swift` / `project.yml`) — a text merge may not be enough; follow the app’s dependency notes in `AGENTS.md`.

### 5. Prove the fork still works

Inside the submodule, as far as this environment allows:

- Regenerate project if needed (`xcodegen` on iOS after `app.yml` / target YAML conflicts).
- Build and run unit tests when tooling is available (`swift run tools ci unit-tests` on iOS host/CI patterns).
- Re-check decision-tagged tests for surfaces the merge touched.

Xcode / simulator / snapshots: if this session is the Linux container, hand off to the Mac host (`/verify-decision` for decisions whose UI moved). Do not claim `done` on decisions from an upstream sync alone.

Do **not** push or open a PR unless the user asks.

### 6. Land on the fork, then pin meta

When the user wants it landed:

1. Ensure the merge is on app **`develop`**, then `git push origin develop`. No PR / feature branch required for the solo trunk. Do not update app `main` as part of an upstream sync.
2. On **meta `develop`**, update the submodule to that app `develop` SHA, then stage **only** the gitlink:

```bash
cd /workspace   # meta root
git checkout develop
git pull origin develop
# submodule at the landed app develop SHA
git add ios     # or android — stages the pin only
```

3. Commit the pin on meta **`develop` only if the user asks** (Conventional Commits, e.g. `chore(ios): pin after upstream merge`). Never commit secrets or stage `ios/**` file paths as normal meta files. Do not advance meta `main` during a sync.

After an old meta checkout: `git submodule update`.

## Cadence (advice to the user, not a hard gate)

Sync regularly (e.g. weekly or per Element X release). Long gaps explode conflicts on identity, Compound, coordinators, and SDK pins.

## Out of scope

- Authoring / amending decisions for new product → `/decision`
- Feature implementation of open decisions → `/implement-decision`
- Host Xcode verification of a decision → `/verify-decision`
- Merging Element X into meta or vendoring upstream trees into `specs/`
- Force-push / history rewrite of the fork’s `develop`

## Multi-app

Same process per submodule. One product design in `specs/`; each platform replays. Pin each app independently on meta.
