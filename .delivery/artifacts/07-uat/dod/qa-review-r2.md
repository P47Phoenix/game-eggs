# QA Regression Check — Rainbow Six 3: Raven Shield Egg (R2)

**Date:** 2026-04-11
**Reviewer:** QA Engineer (automated regression)
**File:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Checks

| # | Check | Expected | Actual | Result |
|---|-------|----------|--------|--------|
| 1 | JSON valid (no parse errors) | valid | valid | PASS |
| 2 | NAME `default_value` | `"Pterodactyl Raven Shield"` | `"Pterodactyl Raven Shield"` | PASS |
| 3 | MAX_PLAYERS `default_value` | `"16"` | `"16"` | PASS |
| 4 | MAP_2 `default_value` | `"Shipyard"` | `"Shipyard"` | PASS |
| 5 | MAP_0 `rules` (no regression) | `"required\|string\|max:64"` | `"required\|string\|max:64"` | PASS |
| 6 | GAMETYPE_0 `rules` (no regression) | `"required\|string\|max:128"` | `"required\|string\|max:128"` | PASS |
| 7 | INSTALL_OPENRVS `user_editable` (no regression) | `true` | `true` | PASS |
| 8 | GAME_PRESET `default_value` unchanged | `"COOP"` | `"COOP"` | PASS |
| 9 | PORT `default_value` unchanged | `"7777"` | `"7777"` | PASS |
| 10 | `startup` field unchanged | `"/entrypoint.sh"` | `"/entrypoint.sh"` | PASS |
| 11 | `done` string unchanged | `"OpenRVS is up to date"` | `"OpenRVS is up to date"` | PASS |

---

## Verdict

**DONE** — All 11 checks PASS. No regressions detected. The three targeted changes (NAME, MAX_PLAYERS, MAP_2 defaults) are correctly applied and all surrounding fields are unaffected.
