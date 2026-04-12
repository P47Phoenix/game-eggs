# QA DoD Validation Review — Stage 6 Implementation
# Rainbow Six 3: Raven Shield — Egg JSON & README

**Date:** 2026-04-11
**Validator:** QA Engineer (DoD Validator)
**Stage:** 6 — Dev Implementation
**Egg File:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**README:** `rainbow_six_3_raven_shield/README.md`
**Acceptance Criteria Source:** `.delivery/artifacts/05-plan/po/stories.md`

---

## Gate Criteria Results

| # | Gate Criterion | Result | Detail |
|---|---|---|---|
| G1 | 6 MAP slots and 6 GAMETYPE slots present | **PASS** | MAP_0–MAP_5 and GAMETYPE_0–GAMETYPE_5 all present in `variables` array. |
| G2 | Slot 1–5 map descriptions contain full inline list of all 18 valid map names (not a cross-reference) | **PASS** | All of MAP_1–MAP_5 descriptions contain inline list ending with "Warehouse"; no cross-reference found. |
| G3 | Slot 1–5 gametype descriptions contain all 6 valid game type values with human-readable labels (not a cross-reference) | **PASS** | All of GAMETYPE_1–GAMETYPE_5 contain all 6 values with labels; no cross-reference found. |
| G4 | All GAMETYPE_* rules have max:128 | **PASS** | GAMETYPE_0 = `required\|string\|max:128`; GAMETYPE_1–5 = `nullable\|string\|max:128`. No max:64 present. |
| G5 | MAP_0 and GAMETYPE_0 are required; all others nullable | **PASS** | MAP_0 = `required\|string\|max:64`; GAMETYPE_0 = `required\|string\|max:128`; MAP_1–5 and GAMETYPE_1–5 all begin with `nullable`. |
| G6 | docker_images key = value = full URI | **PASS** | Key and value are both `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` (forward-slash escaping is valid JSON). |
| G7 | Egg description includes /rvs operator warning | **CONDITIONAL PASS** | Warning is present (describes permission denied crash, root cause, and bind-mount workaround). However, AC-B2-02 requires the warning to appear *before* any other content. The current description places the IMPORTANT warning after the game prose. See Finding F-01. |
| G8 | README Known Issues section covers root cause and workaround | **PASS** | Section "Known Issues / Operator Requirements → /rvs Volume Mount Requirement" states exit code 1 crash, hardcoded /rvs root cause, and Wings bind-mount workaround. |

**Overall Gate Result: 7 PASS / 0 HARD FAIL — 1 ADVISORY finding on warning placement (G7)**

---

## Detailed Acceptance Criteria Verification

### Track A — Map Description Inline Content (AC-A-01 to AC-A-06)

| AC | Variable | Result | Note |
|---|---|---|---|
| AC-A-01 | MAP_1 | PASS | Contains all 18 map names including "Warehouse"; no "See MAP_0" phrase. |
| AC-A-02 | MAP_2 | PASS | Same inline list as MAP_1. |
| AC-A-03 | MAP_3 | PASS | Same inline list. |
| AC-A-04 | MAP_4 | PASS | Same inline list. |
| AC-A-05 | MAP_5 | PASS | Same inline list. |
| AC-A-06 | MAP_0 | PASS | Inline list present; no regression. |

All 18 required map names verified present in MAP_0–MAP_5:
Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse.

### Track A — Gametype Description Inline Content (AC-A-07 to AC-A-12)

| AC | Variable | Result | Note |
|---|---|---|---|
| AC-A-07 | GAMETYPE_1 | PASS | All 6 values with labels; no "See GAMETYPE_0" phrase. |
| AC-A-08 | GAMETYPE_2 | PASS | Same inline list. |
| AC-A-09 | GAMETYPE_3 | PASS | Same inline list. |
| AC-A-10 | GAMETYPE_4 | PASS | Same inline list. |
| AC-A-11 | GAMETYPE_5 | PASS | Same inline list. |
| AC-A-12 | GAMETYPE_0 | PASS | All 6 values with labels; no regression. |

All 6 required gametype values verified present in GAMETYPE_0–GAMETYPE_5:
`R6Game.R6TerroristHuntCoopGame`, `R6Game.R6TeamBomb`, `R6Game.R6HostageRescueAdvGame`, `R6Game.R6TeamDeathMatchGame`, `R6Game.R6EscortPilotGame`, `R6Game.R6DeathMatch`.

### Track A — GAMETYPE Rules Normalisation (AC-A-13 to AC-A-18)

| AC | Variable | Actual Rules | Result |
|---|---|---|---|
| AC-A-13 | GAMETYPE_0 | `required\|string\|max:128` | PASS |
| AC-A-14 | GAMETYPE_1 | `nullable\|string\|max:128` | PASS |
| AC-A-15 | GAMETYPE_2 | `nullable\|string\|max:128` | PASS |
| AC-A-16 | GAMETYPE_3 | `nullable\|string\|max:128` | PASS |
| AC-A-17 | GAMETYPE_4 | `nullable\|string\|max:128` | PASS |
| AC-A-18 | GAMETYPE_5 | `nullable\|string\|max:128` | PASS |

### Track A — Slot Required/Nullable (AC-A-19 to AC-A-22)

| AC | Check | Actual | Result |
|---|---|---|---|
| AC-A-19 | MAP_0 begins with `required` | `required\|string\|max:64` | PASS |
| AC-A-20 | GAMETYPE_0 begins with `required` | `required\|string\|max:128` | PASS |
| AC-A-21 | MAP_1–MAP_5 begin with `nullable` | All = `nullable\|string\|max:64` | PASS |
| AC-A-22 | GAMETYPE_1–GAMETYPE_5 begin with `nullable` | All = `nullable\|string\|max:128` | PASS |

### Track A — Rotation Slot Expansion (AC-A-23 to AC-A-28)

| AC | Check | Actual | Result |
|---|---|---|---|
| AC-A-23 | MAP_3, MAP_4, MAP_5 present in variables | Present | PASS |
| AC-A-24 | GAMETYPE_3, GAMETYPE_4, GAMETYPE_5 present | Present | PASS |
| AC-A-25 | MAP_3–5 have `default_value: ""` and `nullable` | default_value="" and rules begin with nullable | PASS |
| AC-A-26 | GAMETYPE_3–5 have `default_value: ""` and `nullable` | default_value="" and rules begin with nullable | PASS |
| AC-A-27 | All new slots have `user_viewable: true` and `user_editable: true` | Confirmed for all 6 new slots | PASS |
| AC-A-28 | All new slots have `field_type: "text"` | Confirmed for all 6 new slots | PASS |

### Track A — AC-A-29: Invalid GAMETYPE Error Warning in Descriptions

| AC | Check | Result | Detail |
|---|---|---|---|
| AC-A-29 | All GAMETYPE_* descriptions warn that invalid values cause entrypoint exit with error at startup | **FAIL** | No GAMETYPE field description (slots 0–5) contains a warning that invalid values will cause the server to exit with an error at startup. The descriptions list valid values but do not include the required "invalid values cause startup failure / entrypoint exit" warning text. |

### Track A — Structural Integrity (AC-A-30 to AC-A-33)

| AC | Check | Result | Detail |
|---|---|---|---|
| AC-A-30 | JSON valid (`python -m json.tool` exits 0) | PASS | Confirmed. |
| AC-A-31 | `meta.version` = `"PTDL_v2"` | PASS | Confirmed at line 4. |
| AC-A-32 | `docker_images` key = value = `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` | PASS | Both key and value equal full URI. |
| AC-A-33 | Unmodified variables unchanged | PASS | GAME_PRESET, PORT, NAME, MAX_PLAYERS, ADMIN_PASSWORD, GAME_PASSWORD, INTERNET_SERVER, INSTALL_OPENRVS all appear structurally intact. |

### Track B2 — /rvs Crash Warning (AC-B2-01 to AC-B2-04)

Track B2 applies (no override variable exists; Gate 1 = no override).

| AC | Check | Result | Detail |
|---|---|---|---|
| AC-B2-01 | Description contains operator warning: (a) crash on startup, (b) root cause (/rvs hardcoded, uid 1000), (c) image-layer defect, (d) bind-mount workaround | PARTIAL PASS | (a) Exit code 1 and permission denied mentioned. (b) Root cause described (Wings mounts at /home/container, image hardcodes /rvs). (c) "image-layer defect" not explicitly stated but implied. (d) Wings bind-mount workaround described. All four points are substantially covered. |
| AC-B2-02 | Warning placed *before* any other content in the description field | **FAIL** | The description begins with game description prose ("Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter..."), followed by the IMPORTANT warning. AC-B2-02 requires the warning to appear first. |
| AC-B2-03 | Description does not claim the crash has been fixed | PASS | No such claim present. |
| AC-B2-04 | README has dedicated Known Issue section: (a) crash+exit code 1, (b) root cause, (c) workaround | PASS | "Known Issues / Operator Requirements → /rvs Volume Mount Requirement" covers all three sub-points. |

---

## Test Case Results (TC-01 to TC-09)

| TC | Description | Result | Note |
|---|---|---|---|
| TC-01 | MAP_1–5 contain "Warehouse"; no "See MAP_0" | PASS | All five contain "Warehouse". No cross-reference phrase detected. |
| TC-02 | GAMETYPE_1–5 contain "R6Game.R6DeathMatch" and "Deathmatch"; no "See GAMETYPE_0" | PASS | All five contain both strings. No cross-reference phrase detected. |
| TC-03 | All GAMETYPE_0–5 rules contain `max:128` and none contain `max:64` | PASS | Confirmed. |
| TC-04 | MAP_0 and GAMETYPE_0 begin with `required`; MAP_1–5 and GAMETYPE_1–5 begin with `nullable` | PASS | Confirmed. |
| TC-05 | MAP_3, MAP_4, MAP_5, GAMETYPE_3, GAMETYPE_4, GAMETYPE_5 all present in variables array | PASS | All 6 confirmed present. |
| TC-06 | New slots have `default_value=""`, `user_viewable: true`, `user_editable: true`, `field_type: "text"` | PASS | All 6 new slots pass all four checks. |
| TC-07 | GAMETYPE descriptions warn invalid values cause entrypoint exit (static check) | **FAIL** | No GAMETYPE slot description contains this warning text. AC-A-29 unmet. |
| TC-08 | `python -m json.tool` exits 0 | PASS | Confirmed. |
| TC-09 | Unmodified variables are intact | PASS | All unmodified variables appear unchanged. |

**TC results: 8 PASS / 2 FAIL (TC-07 and indirect effect of B2-02 on TC-11 equivalent)**

---

## Findings Summary

### F-01 (FAIL) — AC-B2-02: Operator warning not placed first in egg description

**Criterion:** AC-B2-02 — The warning in the `description` field must be placed before any other content so it is visible at the top of the panel's egg description display.

**Actual:** The description begins with the game prose ("Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter released in 2003..."), and the IMPORTANT warning follows after approximately 2 sentences.

**Required fix:** Move the IMPORTANT /rvs operator warning text to the very beginning of the `description` field value, before any game description prose.

**Severity:** BLOCKING — This is a DoD acceptance criterion (AC-B2-02), not advisory.

---

### F-02 (FAIL) — AC-A-29: GAMETYPE descriptions do not warn about invalid-value startup exit

**Criterion:** AC-A-29 — The egg description for all GAMETYPE slots (0–5) must warn that invalid values cause the server to exit with an error at startup (confirmed by `validate_gametype` function in the entrypoint).

**Actual:** All six GAMETYPE descriptions list valid values with human-readable labels but contain no text warning that an invalid value causes the entrypoint to exit with a non-zero code before launching Wine.

**Required fix:** Append a warning to the description of every GAMETYPE_* variable, e.g.: "An invalid value will cause the server to exit with an error at startup before any game process is launched."

**Severity:** BLOCKING — This is a DoD acceptance criterion (AC-A-29), not advisory.

---

## Overall Verdict

**STATUS: NOT_DONE — 2 blocking failures must be resolved before merge.**

| Criterion | Result |
|---|---|
| 6 MAP + 6 GAMETYPE slots | PASS |
| MAP_1–5 full inline map list | PASS |
| GAMETYPE_1–5 full inline gametype list | PASS |
| All GAMETYPE_* max:128 | PASS |
| MAP_0/GAMETYPE_0 required; 1–5 nullable | PASS |
| docker_images key = value = full URI | PASS |
| Egg description includes /rvs warning | PASS (warning present but placed incorrectly — AC-B2-02 FAIL) |
| README Known Issues covers root cause + workaround | PASS |
| **AC-A-29 GAMETYPE invalid-value warning** | **FAIL** |
| **AC-B2-02 warning placed first in description** | **FAIL** |

*Generated by QA DoD Validator — 2026-04-11*
