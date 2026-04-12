# PO DoD Validation — Round 2
**Date:** 2026-04-11
**Validator:** Michael Connelly (Product Owner)
**Artifact:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**Round:** R2 (post-fix review)

---

## Blocking Criteria Checklist

| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | `NAME` default_value = `"Pterodactyl Raven Shield"` | PASS | JSON line 55: `"default_value": "Pterodactyl Raven Shield"` |
| 2 | `MAX_PLAYERS` default_value = `"16"` | PASS | JSON line 65: `"default_value": "16"` |
| 3 | `MAP_2` default_value = `"Shipyard"` | PASS | JSON line 115: `"default_value": "Shipyard"` |
| 4 | Docker image = `ghcr.io/danpowell88/ravenshield_dedicatedserver` | PASS | JSON lines 13–14: correct key and value |
| 5 | 3-port formula documented (PORT, PORT+1000, PORT+2000) | PASS | README port table and JSON `PORT` description both document the formula |
| 6 | `GAME_PRESET` variable present | PASS | JSON env_variable `GAME_PRESET` at index 0 |
| 7 | `PORT` variable present | PASS | JSON env_variable `PORT` at index 1 |
| 8 | `NAME` variable present | PASS | JSON env_variable `NAME` at index 2 |
| 9 | `MAX_PLAYERS` variable present | PASS | JSON env_variable `MAX_PLAYERS` at index 3 |
| 10 | `MAP_0` variable present | PASS | JSON env_variable `MAP_0` at index 4 |
| 11 | `MAP_1` variable present | PASS | JSON env_variable `MAP_1` at index 6 |
| 12 | `MAP_2` variable present | PASS | JSON env_variable `MAP_2` at index 8 |
| 13 | `GAMETYPE_0` variable present | PASS | JSON env_variable `GAMETYPE_0` at index 5 |
| 14 | `GAMETYPE_1` variable present | PASS | JSON env_variable `GAMETYPE_1` at index 7 |
| 15 | `GAMETYPE_2` variable present | PASS | JSON env_variable `GAMETYPE_2` at index 9 |
| 16 | Author = `"michaelconne@gmail.com"` | PASS | JSON line 9: `"author": "michaelconne@gmail.com"` |
| 17 | Game directory README.md exists and is readable | PASS | File read successfully; all required sections present |
| 18 | Root README.md contains Rainbow Six 3: Raven Shield entry | PASS | Root README line 257: `#### [Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)` — entry is present and in alphabetical order |

---

## Non-Blocking Observations

- The game directory README still shows stale default values in the environment variable table: `NAME` default is documented as `My RavenShield Server` (should be `Pterodactyl Raven Shield`), `MAX_PLAYERS` default is documented as `8` (should be `16`), and `MAP_2` default is documented as `Bank` (should be `Shipyard`). The JSON is correct; only the README table is out of sync. These are documentation discrepancies and do not block import or runtime behaviour, but should be corrected before the PR is merged.

---

## Verdict

**FINAL VERDICT: DONE**

All 18 blocking DoD criteria PASS. The three targeted fixes (`NAME`, `MAX_PLAYERS`, `MAP_2` defaults) are correctly applied in the egg JSON. All required variables are present, the Docker image is correct, the author email is correct, the game directory README exists, and the root README entry is present and alphabetically ordered. The egg is ready to merge subject to the non-blocking README table correction noted above.
