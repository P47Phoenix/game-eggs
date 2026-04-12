# DoD Validator Review — Rainbow Six 3: Raven Shield
**Date:** 2026-04-11
**Reviewer role:** Developer (DoD VALIDATOR)
**Round:** R2

---

## Blocking Criteria

| # | Criterion | Result | Notes |
|---|-----------|--------|-------|
| 1 | Egg JSON is valid PTDL_v2 (`meta.version = "PTDL_v2"`, all required top-level fields present) | PASS | `meta.version`, `name`, `author`, `description`, `features`, `docker_images`, `file_denylist`, `startup`, `config`, `scripts`, `variables` all present |
| 2 | `startup = "/entrypoint.sh"` | PASS | Value is `"\/entrypoint.sh"` (valid JSON escape for `/entrypoint.sh`) |
| 3 | `docker_images` key = value = `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` | PASS | Both key and value resolve to `ghcr.io/danpowell88/ravenshield_dedicatedserver` |
| 4 | All 14 env vars present with correct `env_variable` names | PASS | `GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `MAP_0`, `GAMETYPE_0`, `MAP_1`, `GAMETYPE_1`, `MAP_2`, `GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS` — 14 confirmed |
| 5 | `config.startup` done string = `"OpenRVS is up to date"` | PASS | `"done": "OpenRVS is up to date"` |
| 6 | `config.stop = "^C"` | PASS | Exact match |
| 7 | `author = "michaelconne@gmail.com"` | PASS | Exact match |
| 8 | README.md exists in game directory | PASS | `/rainbow_six_3_raven_shield/README.md` present and complete |
| 9 | Root README contains entry for Rainbow Six 3: Raven Shield | PASS | `#### [Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)` at line 257 |

---

## Verdict

**PASS** — All 9 blocking criteria satisfied, no failures.

**Final verdict: DONE**
