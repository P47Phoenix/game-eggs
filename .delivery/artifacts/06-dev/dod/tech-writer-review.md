# Technical Writer DoD Validation — Stage 6 Round 2 Re-Validation
**Date:** 2026-04-11
**Reviewer role:** Technical Writer / DoD Validator
**Task:** Re-validate Stage 6 documentation (round 2). Verify no regressions and that description leads with operator warning.
**Files reviewed:**
- `rainbow_six_3_raven_shield/README.md`
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Gate Criteria Evaluation

### Criterion 1 — README Known Issues section clearly describes symptom (exit code 1, no console output)

**Result: PASS**

The README section "Known Issues / Operator Requirements > /rvs Volume Mount Requirement" opens with:

> "The container crashes immediately on startup with exit code 1 and produces no console output (or a brief 'permission denied' message before exiting)."

Both required symptoms are named explicitly: the exit code (1) and the absence of console output. The parenthetical covers the edge case where a brief message does appear, which prevents operator confusion about whether the symptom matches.

---

### Criterion 2 — Root cause explained in plain language

**Result: PASS**

The README states:

> "The `GAMEFILES_DIR` path is hardcoded to `/rvs/gamefiles` inside the Docker image. Pterodactyl Wings mounts server data at `/home/container` by default. Because the container process expects to write game files to `/rvs` but Wings places the data volume at `/home/container`, the entrypoint fails immediately with a permission denied error on `/rvs`."

The explanation does not require Docker expertise to follow. It names the conflicting paths, identifies who owns each path (the image vs. Wings), and states the outcome (permission denied). The NOTE callout correctly attributes the issue to the third-party image, not the egg, preventing mis-directed bug reports.

---

### Criterion 3 — Workaround is actionable

**Result: PASS**

The README fix instruction reads:

> "Configure your Wings node to bind-mount the server data directory to `/rvs` instead of `/home/container`. This is done at the Wings server level, not in the Pterodactyl panel egg configuration."

This is actionable: it names the specific configuration layer (Wings node), the specific change required (bind-mount to `/rvs`), and explicitly rules out the panel egg configuration as the location, preventing operators from looking in the wrong place.

Note from R1 review: a link to Wings mount documentation or a configuration snippet would strengthen this further, but the criterion requires actionability, not completeness of procedural steps. The criterion is met.

---

### Criterion 4 — Egg description operator warning is first and understandable to a panel admin

**Result: PASS**

The egg `description` field begins:

> "IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions."

The warning is the first content in the description field — verified by reading the raw JSON. The game description follows after the warning, not before it. No regression from R1.

The warning is understandable to a panel admin with no image internals knowledge: it names the symptom (crash, exit code 1), the cause (path mismatch), and the fix (bind-mount to /rvs) in plain terms. The `IMPORTANT:` prefix is recognisable as a priority marker in panel UI description text.

---

### Criterion 5 — Variable descriptions for MAP_1–5 and GAMETYPE_1–5 are self-contained (no cross-references)

**Result: PASS**

Verified all ten variables (MAP_1 through MAP_5, GAMETYPE_1 through GAMETYPE_5):

**MAP_1 through MAP_5:** Each description reads:
> "Map name for rotation slot N. Valid maps: Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse"

The full map list is embedded inline. No phrases such as "see map list below", "see MAP_0", or any reference to another field or external document.

**GAMETYPE_1 through GAMETYPE_5:** Each description provides the full list of valid game type values with plain-English labels inline:
> "Valid values: R6Game.R6TerroristHuntCoopGame — Terrorist Hunt (Co-op), R6Game.R6TeamBomb — Team Bomb, R6Game.R6HostageRescueAdvGame — Hostage Rescue, R6Game.R6TeamDeathMatchGame — Team Deathmatch, R6Game.R6EscortPilotGame — Escort the Pilot, R6Game.R6DeathMatch — Deathmatch. An invalid game type will cause the server to exit on startup with an error."

No cross-references. Memory lesson applied: PTDL_v2 has no dropdown field type; valid values must be in description text. This constraint is correctly satisfied for all slots 1–5.

---

## Regression Check

No regressions were identified. All criteria satisfied in R1 remain satisfied:
- Variable count (14 variables documented)
- Port documentation (all 3 UDP ports, derived port formula)
- First-run download behavior (archive.org, ~500 MB)
- Valid GAME_PRESET values (all 7 presets)
- Valid map names (18 maps, case-sensitivity warning)
- Valid game type values (6 types, case-sensitivity warning)

---

## Summary

| # | Criterion | Result |
|---|-----------|--------|
| 1 | README Known Issues describes symptom (exit code 1, no console output) | PASS |
| 2 | Root cause explained in plain language | PASS |
| 3 | Workaround is actionable | PASS |
| 4 | Egg description operator warning is first and understandable | PASS |
| 5 | MAP_1–5 and GAMETYPE_1–5 variable descriptions are self-contained | PASS |

**Overall verdict: PASS — all 5 gate criteria satisfied, no regressions detected.**
