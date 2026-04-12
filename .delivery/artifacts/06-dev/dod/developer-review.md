# Developer DoD Review — Stage 6 Re-Validation (R3)

**Date:** 2026-04-11
**Reviewer role:** Developer
**Round:** R3 (re-validation after two targeted fixes)
**Artifacts reviewed:**
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Gate Criteria Results

| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | All 6 MAP slots (MAP_0–MAP_5) present with correct rules | **PASS** | Variables array contains `MAP_0` through `MAP_5`; rules verified per slot. |
| 2 | All 6 GAMETYPE slots (GAMETYPE_0–GAMETYPE_5) present with correct rules | **PASS** | Variables array contains `GAMETYPE_0` through `GAMETYPE_5`; rules verified per slot. |
| 3 | MAP_0/GAMETYPE_0 required; MAP_1–5/GAMETYPE_1–5 nullable | **PASS** | MAP_0: `required\|string\|max:64`; GAMETYPE_0: `required\|string\|max:128`; MAP_1–5: `nullable\|string\|max:64`; GAMETYPE_1–5: `nullable\|string\|max:128`. |
| 4 | All GAMETYPE_* have max:128 | **PASS** | All six GAMETYPE slots (0–5) carry `max:128` in their rules string. No max:64 outliers. |
| 5 | docker_images key = value = full URI | **PASS** | Key: `ghcr.io/danpowell88/ravenshield_dedicatedserver` = Value: `ghcr.io/danpowell88/ravenshield_dedicatedserver`. Both are full URI. |
| 6 | JSON is valid PTDL_v2 | **PASS** | `meta.version` = `"PTDL_v2"`. All required top-level keys present and well-formed. |
| 7 | Description starts with /rvs operator warning | **PASS** | Description begins: "IMPORTANT: This image requires the game data volume to be mounted at /rvs..." — warning is at first position. Fix (1) confirmed applied. |
| 8 | All GAMETYPE_* descriptions include invalid-value exit warning | **PASS** | All six GAMETYPE descriptions (0–5) end with: "An invalid game type will cause the server to exit on startup with an error." Fix (2) confirmed applied across all slots. |

---

## Overall Verdict: ALL GATES PASS

---

## Detailed Notes

### Fix (1) — /rvs Operator Warning at First Position

The egg description field now opens with the operator warning before any game narrative text:

> "IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions."

This is confirmed at the very start of the description string. Gate 7 PASS.

### Fix (2) — Invalid-Value Exit Warning in All GAMETYPE_* Descriptions

All six GAMETYPE variables contain the sentence:

> "An invalid game type will cause the server to exit on startup with an error."

Verified for GAMETYPE_0 through GAMETYPE_5 individually. Gate 8 PASS.

### Structural Completeness

- All 6 MAP slots and 6 GAMETYPE slots are present and interleaved correctly (MAP_0, GAMETYPE_0, MAP_1, GAMETYPE_1, ..., MAP_5, GAMETYPE_5).
- MAP/GAMETYPE slot 0 use `required`; slots 1–5 use `nullable` — consistent with memory lesson.
- MAP_3–5 and GAMETYPE_3–5 have blank default values, which is correct for optional rotation slots.
- GAMETYPE max:128 accommodates longest known game type string (`R6Game.R6TerroristHuntCoopGame` = 31 chars).

### docker_images

Key equals value, both equal the full container image URI. No shorthand or mismatched values. Compliant with memory lesson.

### PTDL_v2 Validity

All required top-level keys present: `_comment`, `meta`, `exported_at`, `name`, `author`, `description`, `features`, `docker_images`, `file_denylist`, `startup`, `config`, `scripts`, `variables`. JSON is well-formed with no syntax issues observed.
