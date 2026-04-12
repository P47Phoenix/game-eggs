# QA DoD Validation — Rainbow Six 3: Raven Shield Egg (Round 2)

**File:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**Validator:** QA Engineer (DoD Validator)
**Date:** 2026-04-11

---

## Blocking Criteria Checks

| # | Criterion | Result | Notes |
|---|-----------|--------|-------|
| 1 | JSON parses without error | PASS | File is valid JSON; no syntax errors detected. |
| 2 | `meta.version` = `"PTDL_v2"` | PASS | `"version": "PTDL_v2"` present. |
| 3 | All required top-level fields present | PASS | `_comment`, `meta`, `exported_at`, `name`, `author`, `description`, `features`, `docker_images`, `file_denylist`, `startup`, `config`, `scripts`, `variables` all present. |
| 4 | All 14 env vars present with non-empty descriptions and sensible defaults | PASS | 14 variables found: `GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `MAP_0`, `GAMETYPE_0`, `MAP_1`, `GAMETYPE_1`, `MAP_2`, `GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`. All have non-empty descriptions. Defaults are sensible (empty string allowed for optional password fields). |
| 5 | `GAME_PRESET` default = `"COOP"`, rules include all valid values | PASS | `default_value: "COOP"`; rules: `in:COOP,ADVERSARIAL,DEATHMATCH,TEAMDEATHMATCH,BOMB,HOSTAGERESCUE,ESCORTPILOT` — all 7 valid presets present. |
| 6 | `PORT` default = `"7777"` | PASS | `default_value: "7777"` confirmed. |
| 7 | `NAME` default is a non-empty string | PASS | `default_value: "My RavenShield Server"` — non-empty. |
| 8 | `MAP_0/1/2` defaults are valid map names (Airport, Alpines, Bank) | PASS | `MAP_0="Airport"`, `MAP_1="Alpines"`, `MAP_2="Bank"` — all match required defaults. |
| 9 | `GAMETYPE_0/1/2` defaults are valid gametype strings | PASS | All three default to `"R6Game.R6TerroristHuntCoopGame"`, which is listed in the valid values for `GAMETYPE_0`. |
| 10 | `scripts.installation.script` is a non-empty string | PASS | Script contains `#!/bin/bash` stub with echo and exit 0 — non-empty. |

---

## Summary

All 10 blocking criteria: **PASS**

No failures detected. The egg file is structurally sound and meets all DoD requirements.

---

## Final Verdict

**DONE**
