# Technical Writer DoD Validation — R2
**Date:** 2026-04-11
**Reviewer role:** Technical Writer / DoD Validator
**Files reviewed:**
- `rainbow_six_3_raven_shield/README.md`
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Blocking Criteria Checklist

| # | Criterion | Result | Notes |
|---|-----------|--------|-------|
| 1 | README documents all 14 environment variables | PASS | All 14 variables present in the Environment Variables table: `GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `MAP_0`, `GAMETYPE_0`, `MAP_1`, `GAMETYPE_1`, `MAP_2`, `GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`. Matches egg JSON exactly. |
| 2 | Port requirements documented with all 3 ports (7777, 8777, 9777) and their purposes | PASS | Server Ports table explicitly lists PORT=7777 (Primary game traffic), PORT+1000=8777 (Server beacon / OpenRVS / GameSpy), PORT+2000=9777 (Beacon port). Warning block reinforces re-allocation requirement if PORT changes. |
| 3 | First-run download behavior mentioned (archive.org) | PASS | Mentioned in Server Overview (line 9) and again in the NOTE callout (line 17): ~500 MB downloaded from archive.org via OpenRVS on first startup. |
| 4 | Valid GAME_PRESET values listed | PASS | All 7 presets listed in both the Environment Variables table and the dedicated Game Presets section with descriptions: `COOP`, `ADVERSARIAL`, `DEATHMATCH`, `TEAMDEATHMATCH`, `BOMB`, `HOSTAGERESCUE`, `ESCORTPILOT`. Matches egg `rules` field exactly. |
| 5 | Valid map names provided as reference | PASS | 18 map names listed in a dedicated Valid Map Names table with a case-sensitivity warning. |
| 6 | Valid GAMETYPE values listed | PASS | 6 game type values listed in a dedicated Valid Game Types table with case-sensitivity warning and plain-English names: `R6Game.R6TerroristHuntCoopGame`, `R6Game.R6TeamBomb`, `R6Game.R6HostageRescueAdvGame`, `R6Game.R6TeamDeathMatchGame`, `R6Game.R6EscortPilotGame`, `R6Game.R6DeathMatch`. |

---

## Additional Observations (Non-blocking)

- Documentation accurately reflects `INSTALL_OPENRVS` as `user_editable: false` in the egg but does not call this out explicitly in the README — minor omission, not blocking.
- The Known Limitations section adds useful operational context not required by the DoD but beneficial for operators.
- Startup done string `"OpenRVS is up to date"` is internal to the egg config and not user-facing; no documentation gap.

---

## Verdict

**PASS** — All 6 blocking criteria are fully satisfied.

**Final verdict: DONE**
