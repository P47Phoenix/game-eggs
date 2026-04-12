# Contribution / Deployment Plan: Tom Clancy's Rainbow Six 3: Raven Shield Egg

**Date:** 2026-04-11
**Author:** Michael Connelly (michaelconne@gmail.com)
**Target Repository:** https://github.com/pterodactyl/game-eggs

---

## 1. Branch Strategy

**Feature branch name:** `feat/rainbow-six-3-raven-shield`

Branch from `main` of the forked repository. All work is committed to this branch; the PR targets `main` of `pterodactyl/game-eggs`.

```
git checkout main
git pull origin main
git checkout -b feat/rainbow-six-3-raven-shield
```

---

## 2. File Change Checklist

### Files to CREATE

| # | Path | Description |
|---|---|---|
| 1 | `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` | PTDL_v2 egg definition (STORY-01) |
| 2 | `rainbow_six_3_raven_shield/README.md` | Game-directory documentation (STORY-02) |

### Files to MODIFY

| # | Path | Change Description |
|---|---|---|
| 3 | `README.md` (root) | Add one row for Raven Shield in alphabetical order (STORY-03) |

### Alphabetical Insertion Point — Root README

The root README uses a heading-per-game list (not a table). The new entry sorts between **Renown** and **SCUM**:

```markdown
#### [Renown](./renown)

#### [Tom Clancy's Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)

#### [SCUM](./scum)
```

> Note: "Rainbow Six" sorts before "Renown" alphabetically (R-a vs R-e), so the entry should actually be placed **before** Renown. Verify exact neighbors at PR time by scanning the rendered README.

Correct alphabetical position: "Rainbow" < "Renown", so the row goes **before** the Renown entry:

```markdown
#### [Tom Clancy's Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)

#### [Renown](./renown)
```

---

## 3. Egg JSON Authoring Checklist (STORY-01 AC coverage)

Before committing `egg-rainbow-six-3-raven-shield.json`, verify each item:

- [ ] `_comment` is exactly `"DO NOT EDIT: FILE GENERATED AUTOMATICALLY BY PTERODACTYL PANEL - PTERODACTYL.IO"`
- [ ] `meta.version` is `"PTDL_v2"` and `meta.update_url` is `null`
- [ ] `name` is `"Raven Shield"` and `author` is `"michaelconne@gmail.com"`
- [ ] `features` is `null` and `file_denylist` is `[]`
- [ ] `docker_images` has exactly one entry: key and value both `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`
- [ ] `startup` is exactly `"/entrypoint.sh"`
- [ ] `config.startup` is `{"done": "OpenRVS is up to date"}`
- [ ] `config.stop` is `"^C"`
- [ ] `variables` array has exactly 14 entries
- [ ] Variable 3 uses `env_variable: "NAME"` (not `"SERVER_NAME"`)
- [ ] `GAME_PRESET` rules include `in:COOP,ADVERSARIAL,DEATHMATCH,TEAMDEATHMATCH,BOMB,HOSTAGERESCUE,ESCORTPILOT`; default `"COOP"`
- [ ] `GAMETYPE_0` default is `"R6Game.R6TerroristHuntCoopGame"`; `GAMETYPE_1` and `GAMETYPE_2` are `nullable`
- [ ] `MAP_0` default `"Airport"`, `MAP_1` default `"Alpines"`, `MAP_2` default `"Bank"` (per STORY-01 AC-01-14); confirm against PRD FR-9 which shows `MAP_1="Bank"`, `MAP_2="Shipyard"` — use PRD FR-9 values as authoritative
- [ ] `PORT` default `"7777"`, rules `"required|numeric"`
- [ ] All `field_type` values are `"text"`
- [ ] No variable has `user_editable: true` with `user_viewable: false`
- [ ] `scripts.installation.container` is `"ghcr.io/ptero-eggs/installers:debian"`
- [ ] `scripts.installation.entrypoint` is `"bash"`
- [ ] `scripts.installation.script` is a no-op bash script with `#!/bin/bash` shebang and an informational echo
- [ ] File is UTF-8, Unix line endings (`\n`)
- [ ] `python -m json.tool egg-rainbow-six-3-raven-shield.json` exits 0

> **Variable default conflict note:** STORY-01 AC-01-14 lists `MAP_1=Alpines`, `MAP_2=Bank`; PRD FR-9 lists `MAP_1=Bank`, `MAP_2=Shipyard`. Use **PRD FR-9** as the authoritative source (`MAP_1="Bank"`, `MAP_2="Shipyard"`). Flag this discrepancy in the PR description if reviewers ask.

---

## 4. README Authoring Checklist (STORY-02 AC coverage)

Before committing `rainbow_six_3_raven_shield/README.md`, verify each item:

- [ ] Server overview section present: names the game, links to `ghcr.io/danpowell88/ravenshield_dedicatedserver`, notes image is an external dependency, states game files download automatically from archive.org on first start with no CD key or Steam login required
- [ ] Installation/setup section present: how to import the egg JSON, how to create a server, note about first-start download delay
- [ ] Environment variable reference table: 14 rows, columns for variable name, default value, allowed/valid values, and description
- [ ] All 6 valid gametype strings listed: `R6Game.R6TerroristHuntCoopGame`, `R6Game.R6TeamBomb`, `R6Game.R6HostageRescueAdvGame`, `R6Game.R6TeamDeathMatchGame`, `R6Game.R6EscortPilotGame`, `R6Game.R6DeathMatch`
- [ ] All 18 valid map names listed: `Airport`, `Alpines`, `Bank`, `Garage`, `Import_Export`, `Island_Dawn`, `MeatPacking`, `Mountain_High`, `Oil_Refinery`, `Parade`, `Peaks`, `Penthouse`, `Presidio`, `Prison`, `Shipyard`, `Streets`, `Training`, `Warehouse`
- [ ] Map names are case-sensitive — note explicitly included
- [ ] Port reference table: 3 rows (7777, 8777, 9777), roles (primary game, server beacon, beacon), derivation formula (`PORT+1000`, `PORT+2000`)
- [ ] Known limitations section covering: (a) image externally maintained; (b) map/gametype values not panel-validated — incorrect values cause startup failure; (c) changing PORT requires re-allocating all 3 derived ports in the panel

---

## 5. PR Checklist (from `.github/pull_request_template.md`)

### Checklist for All Submissions

- [ ] Contribution guidelines reviewed at https://github.com/Ptero-Eggs/.github/blob/main/profile/CONTRIBUTING.md
- [ ] No other open PRs exist for the same egg
- [ ] Changes tested and reviewed with confidence everything works
- [ ] PR submitted from feature branch `feat/rainbow-six-3-raven-shield`, not from `main`

### Egg Update Items

- [ ] **Start command does not use a shell script** — `startup` is `/entrypoint.sh` which is baked into the Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver` (it is part of the image yolk, not a user-managed file). This satisfies the requirement. Explain this in the PR description.
- [ ] **Egg was exported from the panel** — export the egg JSON from a live Pterodactyl panel instance after import, then replace the file with the exported version to ensure `exported_at` timestamp and formatting are panel-generated.

### New Egg Submission Items

1. - [ ] **Server is connectable** — must be verified by starting a server with default variable values and confirming UDP ports 7777, 8777, 9777 are reachable and the panel console logs `OpenRVS is up to date`
2. - [ ] **Custom Docker image justification** — `ghcr.io/danpowell88/ravenshield_dedicatedserver` is required because the game is not on Steam; there is no generic pterodactyl image that can install or run Raven Shield. The image bundles the entrypoint, OpenRVS patch tooling, and `crudini`. Note: this is an externally maintained image. Explain in PR description why a generic image cannot be used and link to the source repo.
   - [ ] Generic image attempted/considered — document that no generic image can serve this game
   - [ ] PR to add yolk not required — entrypoint is already in the external image
3. - [ ] **Root README.md updated** with new entry in alphabetical order linking to `rainbow_six_3_raven_shield/`
4. - [ ] **Game-directory README.md added** at `rainbow_six_3_raven_shield/README.md`
5. - [ ] **Start command does not use a shell script** (repeated) — `/entrypoint.sh` is image-baked
6. - [ ] **Egg exported from panel** (repeated) — use panel-exported JSON

---

## 6. Rollback Plan

If the PR is rejected, the following actions are required depending on the rejection reason:

| Rejection Reason | Required Fix |
|---|---|
| Startup command (`/entrypoint.sh`) not accepted as a yolk | Re-evaluate whether `/entrypoint.sh` can be submitted as a new yolk PR to `Ptero-Eggs/yolks`; alternatively, refactor to use a generic installer image with a user-managed startup wrapper (significant rework) |
| Custom Docker image not accepted — generic image required | Research whether a generic `debian` or `steamcmd` yolk can replicate the entrypoint behaviour; requires building and contributing a new yolk |
| JSON schema non-conformance | Fix the specific schema violations identified by the reviewer; re-export from panel after corrections |
| Variable naming/value issues | Update `env_variable` keys, `rules` strings, or defaults as directed by reviewer; update README env var table to match |
| Directory naming convention incorrect | Rename directory from `rainbow_six_3_raven_shield/` to the convention required (e.g., `rainbow_six/raven_shield/`); update root README link accordingly. Note: PRD FR-1/FR-2 and stories use `rainbow_six_3_raven_shield/` while PRD FR-3/AC-8 reference `rainbow_six/raven_shield/` — resolve this before opening the PR (see Go/No-Go criterion #1 below) |
| README content insufficient | Add or correct the missing sections per reviewer feedback |
| Server connectivity test failed | Debug the Docker image and variable configuration; re-run the test before resubmitting |

**No repository state needs to be reverted on rejection** — the feature branch `feat/rainbow-six-3-raven-shield` is the only change set. Close the PR, apply fixes on the same branch or a new branch, and resubmit.

---

## 7. Go / No-Go Criteria

All of the following must be true before opening the PR:

| # | Criterion | Verification Method |
|---|---|---|
| 1 | **Directory name resolved** — Confirm whether the directory is `rainbow_six_3_raven_shield/` (stories) or `rainbow_six/raven_shield/` (PRD FR-1). Choose one and apply it consistently across all three files. | Manual decision; review existing repo conventions (e.g., `./arma/arma3` vs flat names) |
| 2 | **JSON validates** — `python -m json.tool egg-rainbow-six-3-raven-shield.json` exits 0 | Run locally before commit |
| 3 | **Panel import succeeds** — egg imports into a Pterodactyl v1.x panel instance without error; all 14 variables appear | Manual import test |
| 4 | **Egg re-exported from panel** — the committed JSON is the panel-generated export (satisfies PR checklist item 6) | Export from panel after successful import |
| 5 | **Server boots to ready state** — server started with default variable values logs `OpenRVS is up to date` in the panel console | Manual end-to-end test |
| 6 | **All three UDP ports reachable** — 7777, 8777, 9777 are open and accepting traffic from outside the container | `nmap -sU` or `netcat -u` from host |
| 7 | **No CD key or Steam login required** — first-run download from archive.org completes unattended | Observed during server boots test (criterion 5) |
| 8 | **Root README renders correctly** — new Raven Shield row is alphabetically ordered and Markdown table/heading renders without formatting errors | Visual inspection on GitHub preview or `markdownlint` |
| 9 | **No duplicate README entry** — search root README for "Raven Shield" confirms exactly one occurrence | `grep -c "Raven Shield" README.md` returns `1` |
| 10 | **Feature branch is up to date with upstream main** — rebase onto latest `main` of `pterodactyl/game-eggs` immediately before opening the PR | `git fetch upstream && git rebase upstream/main` |
| 11 | **Custom Docker image justification drafted** — PR description includes explanation of why `ghcr.io/danpowell88/ravenshield_dedicatedserver` is necessary and why no generic image can be used | Review PR description draft |
| 12 | **Author email consistent** — `author` field in egg JSON is `"michaelconne@gmail.com"` matching the GitHub account opening the PR | Code review of JSON |

---

## 8. Dependency Execution Order

```
STORY-01: Author egg-rainbow-six-3-raven-shield.json
    |
    +--> STORY-02: Author rainbow_six_3_raven_shield/README.md
    |
    +--> STORY-03: Update root README.md
    |
    v
Go/No-Go gate: all 12 criteria met
    |
    v
Open PR from feat/rainbow-six-3-raven-shield --> pterodactyl/game-eggs main
```

STORY-02 and STORY-03 can be drafted in parallel once STORY-01 variable names and the directory name are confirmed.
