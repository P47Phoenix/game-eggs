# QA DoD Review — Stage 6 Re-validation (Round 3)

**Date:** 2026-04-11
**Reviewer:** QA Engineer (delivery-team:quality)
**Artifacts reviewed:**
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
- `rainbow_six_3_raven_shield/README.md`
- `.delivery/artifacts/05-plan/po/stories.md`

**Scope:** Re-validation after two targeted fixes:
1. F-01: `/rvs` operator warning moved to first position in egg `description`
2. F-02: All 6 `GAMETYPE_*` descriptions now include invalid-value exit warning sentence

---

## Gate Results

| # | Gate Criterion | Result | Evidence |
|---|----------------|--------|----------|
| 1 | 6 MAP and 6 GAMETYPE slots present | **PASS** | MAP_0–MAP_5 and GAMETYPE_0–GAMETYPE_5 all present in `variables` array |
| 2 | Slot 1–5 map descriptions contain full inline list of all 18 valid map names | **PASS** | MAP_1–MAP_5 each contain "Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse" |
| 3 | Slot 1–5 gametype descriptions contain all 6 values with labels | **PASS** | GAMETYPE_0–GAMETYPE_5 all list all 6 game type values with human-readable labels; no cross-references present |
| 4 | All GAMETYPE_* rules have max:128 | **PASS** | GAMETYPE_0: `required\|string\|max:128`; GAMETYPE_1–5: `nullable\|string\|max:128` |
| 5 | MAP_0/GAMETYPE_0 required; all others nullable | **PASS** | MAP_0 and GAMETYPE_0 use `required`; MAP_1–5 and GAMETYPE_1–5 all use `nullable` |
| 6 | docker_images key = value = full URI | **PASS** | Both key and value resolve to `ghcr.io/danpowell88/ravenshield_dedicatedserver` (forward-slash JSON escaping is valid) |
| 7 | Egg description starts with /rvs operator warning (F-01 fix) | **PASS** | Description opens with "IMPORTANT: This image requires the game data volume to be mounted at /rvs..." before any game prose |
| 8 | All 6 GAMETYPE_* descriptions include invalid-value exit warning (F-02 fix) | **PASS** | All six descriptions end with "An invalid game type will cause the server to exit on startup with an error." |
| 9 | README Known Issues section covers root cause and workaround | **PASS** | README "Known Issues / Operator Requirements" section documents: (a) crash symptom (exit code 1, permission denied), (b) root cause (Docker image hardcodes `/rvs`; Wings mounts at `/home/container`; uid 1000 denied), (c) Wings bind-mount workaround |

---

## Detailed Gate Verification

### Gate 1 — Slot Count
Variables array contains exactly MAP_0, MAP_1, MAP_2, MAP_3, MAP_4, MAP_5 and GAMETYPE_0, GAMETYPE_1, GAMETYPE_2, GAMETYPE_3, GAMETYPE_4, GAMETYPE_5. All 12 rotation-slot variables confirmed present.

### Gate 2 — Map Inline Lists (Slots 1–5)
Each of MAP_1 through MAP_5 contains an identical inline list of 18 map names. Confirmed "Warehouse" (last item per TC-01) present in all five. No "See MAP_0" cross-reference language detected in any slot.

MAP_0 description also contains the full 18-name list — no regression.

### Gate 3 — Gametype Inline Lists (Slots 1–5)
Each of GAMETYPE_0 through GAMETYPE_5 contains all 6 game type values:
- R6Game.R6TerroristHuntCoopGame — Terrorist Hunt (Co-op)
- R6Game.R6TeamBomb — Team Bomb
- R6Game.R6HostageRescueAdvGame — Hostage Rescue
- R6Game.R6TeamDeathMatchGame — Team Deathmatch
- R6Game.R6EscortPilotGame — Escort the Pilot
- R6Game.R6DeathMatch — Deathmatch

Confirmed "R6Game.R6DeathMatch" and "Deathmatch" present in all six (TC-02). No "See GAMETYPE_0" cross-reference language detected.

### Gate 4 — max:128 Rules
All six GAMETYPE_* variables use `max:128`. No `max:64` found in any GAMETYPE field.

### Gate 5 — Required/Nullable
- `MAP_0`: `required|string|max:64` — correct
- `GAMETYPE_0`: `required|string|max:128` — correct
- `MAP_1` through `MAP_5`: all `nullable|string|max:64` — correct
- `GAMETYPE_1` through `GAMETYPE_5`: all `nullable|string|max:128` — correct

### Gate 6 — docker_images Key/Value Parity
```json
"docker_images": {
    "ghcr.io\/danpowell88\/ravenshield_dedicatedserver": "ghcr.io\/danpowell88\/ravenshield_dedicatedserver"
}
```
JSON `\/` escape is semantically identical to `/`. Key equals value equals `ghcr.io/danpowell88/ravenshield_dedicatedserver`. AC-A-32 satisfied.

### Gate 7 — F-01 Fix: Description Opens with /rvs Warning
Description field begins: `"IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions."`

This content appears before the game description prose. AC-B2-02 satisfied.

The description does not claim the startup crash has been fixed (it describes it as a current constraint requiring operator action). AC-B2-03 satisfied.

### Gate 8 — F-02 Fix: Invalid-Value Exit Warning in All 6 GAMETYPE Descriptions
All six GAMETYPE_* descriptions (slots 0 through 5) include the sentence: `"An invalid game type will cause the server to exit on startup with an error."` This satisfies AC-A-29 for all slots.

### Gate 9 — README Known Issues
README section "Known Issues / Operator Requirements" contains:
- **Symptom statement:** "The container crashes immediately on startup with exit code 1 and produces no console output (or a brief 'permission denied' message before exiting)."
- **Root cause:** "The `GAMEFILES_DIR` path is hardcoded to `/rvs/gamefiles` inside the Docker image... Pterodactyl Wings mounts server data at `/home/container` by default. Because the container process expects to write game files to `/rvs` but Wings places the data volume at `/home/container`, the entrypoint fails immediately with a permission denied error on `/rvs`."
- **Workaround:** "Configure your Wings node to bind-mount the server data directory to `/rvs` instead of `/home/container`. This is done at the Wings server level, not in the Pterodactyl panel egg configuration."

AC-B2-04 (a), (b), and (c) all satisfied.

---

## Additional Track A Spot-Checks

| Check | Result |
|-------|--------|
| meta.version = "PTDL_v2" | PASS |
| MAP_3/4/5 default_value = "" | PASS |
| GAMETYPE_3/4/5 default_value = "" | PASS |
| All new slots: user_viewable=true, user_editable=true | PASS |
| All new slots: field_type="text" | PASS |
| Unmodified variables (GAME_PRESET, PORT, NAME, MAX_PLAYERS, etc.) — no regression observed | PASS |

---

## Fix Confirmation

| Fix ID | Description | Prior Status | Current Status |
|--------|-------------|--------------|----------------|
| F-01 | /rvs operator warning at first position in egg description | FAIL | **PASS** |
| F-02 | All 6 GAMETYPE_* descriptions include invalid-value exit warning | FAIL | **PASS** |

---

## Overall Verdict

**ALL GATE CRITERIA: PASS**

Both targeted fixes (F-01, F-02) have been correctly applied. No regressions detected on any previously passing criteria. The implementation satisfies all Stage 6 Definition of Done requirements for Track A and Track B2.

**Stage 6 DoD: COMPLETE — recommend merge approval.**
