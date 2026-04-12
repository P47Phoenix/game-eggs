# Test Strategy: Rainbow Six 3: Raven Shield — Bug Fix & Usability Enhancements

**Date:** 2026-04-11
**Author:** QA Engineer
**Pipeline:** run-2026-04-11-r6fix
**Type:** BUG_FIX (with bundled usability enhancements)
**Stories Version:** 2026-04-11 (STORY-01)
**Target file:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## 1. Acceptance Criteria Testability Review

All ACs in stories.md are judged testable as described below.

| AC | Testability | Method |
|---|---|---|
| AC-A-01 to AC-A-05 | Testable — static | JSON field inspection for description string content |
| AC-A-06 | Testable — static | JSON field inspection; regression baseline comparison |
| AC-A-07 to AC-A-11 | Testable — static | JSON field inspection for all six gametype strings and labels |
| AC-A-12 | Testable — static | JSON field inspection; regression baseline comparison |
| AC-A-13 to AC-A-18 | Testable — static | JSON `rules` substring check for `max:128` |
| AC-A-19 to AC-A-22 | Testable — static | JSON `rules` prefix check for `required` / `nullable` |
| AC-A-23 to AC-A-24 | Testable — static | Presence of six new `env_variable` names in `variables` array |
| AC-A-25 to AC-A-28 | Testable — static | JSON field value checks on new variable objects |
| AC-A-29 | Testable — static (description text) + empirical (Docker, optional) | Description text check; runtime exit-code test with invalid value |
| AC-A-30 | Testable — static | `python -m json.tool` exit code |
| AC-A-31 | Testable — static | `meta.version` field value check |
| AC-A-32 | Testable — static | `docker_images` key and value exact-match check |
| AC-A-33 | Testable — static | Byte-for-byte field comparison against pre-change baseline |
| AC-B1-01 to AC-B1-03 | Testable — static (if B1 active) | JSON field inspection of new variable |
| AC-B1-04 | Testable — empirical (Docker required, if B1 active) | `docker run --user 1000:1000` startup check |
| AC-B1-05 | Testable — static (if B1 active) | Absence of warning text in `description` field |
| AC-B2-01 to AC-B2-03 | Testable — static (if B2 active) | `description` field text content inspection |
| AC-B2-04 | Testable — static (if B2 active) | README section heading and body text inspection |

**Conclusion:** All ACs are fully testable. Track A ACs are testable by static JSON inspection without any running infrastructure. Track B empirical test (AC-B1-04) requires Docker but is explicitly scoped to that sub-track only.

---

## 2. Test Scope

### 2.1 In Scope

**Track A (always):**
- Static JSON validity of the modified egg file
- Variable array completeness: 6 MAP slots (MAP_0–MAP_5) and 6 GAMETYPE slots (GAMETYPE_0–GAMETYPE_5) all present
- Description content verification for every MAP and GAMETYPE slot
- `max:128` rules normalisation for all GAMETYPE fields
- `required` / `nullable` enforcement for slot-0 vs slots 1–5
- New variable object field values (`default_value`, `user_viewable`, `user_editable`, `field_type`)
- `docker_images` key-equals-value exact URI check
- `meta.version` PTDL_v2 check
- Regression check on all unmodified variables
- Error-case: invalid GAMETYPE_0 causes non-zero exit (static description check; Docker test where available)

**Track B (gated — one sub-track only):**
- B1 static: new env var object fields and description text
- B1 empirical: Docker container startup without "Permission denied" crash
- B2 static: warning text present and positioned correctly in `description`
- B2 static: README "Known Issue: Startup Crash" section content

### 2.2 Out of Scope

- Building or modifying the Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver`
- Live Pterodactyl panel import tests
- Network/port reachability testing (UDP 7777/8777/9777)
- Entrypoint script internal logic beyond what is observable from container stdout/exit code
- Other egg files in the repository
- CI/CD pipeline configuration

---

## 3. Test Cases

### Track A — Static JSON Verification

---

#### TC-A-01: Map Slot Description Inline Content

**Covers:** AC-A-01 to AC-A-06, stories.md TC-01

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. Parse the JSON and extract the `description` field for each of `MAP_0`, `MAP_1`, `MAP_2`, `MAP_3`, `MAP_4`, `MAP_5`.
2. For each slot, assert the description contains the literal string `"Warehouse"`.
3. For `MAP_1` through `MAP_5`, assert the description does **not** contain the phrase `"See MAP_0"` or any equivalent cross-reference.
4. For `MAP_0`, assert the description is byte-for-byte identical to the pre-change baseline (no regression).
5. For each of `MAP_1` through `MAP_5`, also assert presence of each of the following map name strings:
   - `Airport`, `Alpines`, `Bank`, `Garage`, `Import_Export`, `Island_Dawn`, `MeatPacking`, `Mountain_High`, `Oil_Refinery`, `Parade`, `Peaks`, `Penthouse`, `Presidio`, `Prison`, `Shipyard`, `Streets`, `Training`, `Warehouse`

**Pass criteria:** All 18 map names present in each of MAP_1–MAP_5 descriptions; no cross-reference phrase; MAP_0 description unchanged.

---

#### TC-A-02: Gametype Slot Description Inline Content

**Covers:** AC-A-07 to AC-A-12, stories.md TC-02

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. Parse the JSON and extract the `description` field for each of `GAMETYPE_0` through `GAMETYPE_5`.
2. For each slot, assert presence of all six gametype class strings:
   - `R6Game.R6TerroristHuntCoopGame`
   - `R6Game.R6TeamBomb`
   - `R6Game.R6HostageRescueAdvGame`
   - `R6Game.R6TeamDeathMatchGame`
   - `R6Game.R6EscortPilotGame`
   - `R6Game.R6DeathMatch`
3. For each slot, assert presence of all six human-readable labels:
   - `Terrorist Hunt Co-op`
   - `Team Bomb`
   - `Hostage Rescue`
   - `Team Deathmatch`
   - `Escort the Pilot`
   - `Deathmatch`
4. For `GAMETYPE_1` through `GAMETYPE_5`, assert the description does **not** contain the phrase `"See GAMETYPE_0"` or any equivalent cross-reference.
5. For `GAMETYPE_0`, assert the description is byte-for-byte identical to the pre-change baseline (no regression).

**Pass criteria:** All six gametype strings and all six labels present in every slot description; no cross-reference; GAMETYPE_0 unchanged.

---

#### TC-A-03: GAMETYPE Rules max:128 Normalisation

**Covers:** AC-A-13 to AC-A-18, stories.md TC-03

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. Parse the JSON and extract the `rules` field for each of `GAMETYPE_0` through `GAMETYPE_5`.
2. For each field, assert the `rules` string contains the substring `max:128`.
3. For each field, assert the `rules` string does **not** contain `max:64`.

**Pass criteria:** All six GAMETYPE `rules` fields contain `max:128`; none contain `max:64`.

---

#### TC-A-04: Slot-0 Required / Slots 1–5 Nullable

**Covers:** AC-A-19 to AC-A-22, stories.md TC-04

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. Parse the JSON. For `MAP_0`, assert `rules` begins with `"required"` (i.e., the first token before the first `|` is `required`).
2. For `GAMETYPE_0`, assert `rules` begins with `"required"`.
3. For `MAP_1`, `MAP_2`, `MAP_3`, `MAP_4`, `MAP_5`, assert each `rules` field begins with `"nullable"`.
4. For `GAMETYPE_1`, `GAMETYPE_2`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`, assert each `rules` field begins with `"nullable"`.
5. Confirm no slot-0 field begins with `nullable`.
6. Confirm no slot-1+ field begins with `required`.

**Pass criteria:** MAP_0 and GAMETYPE_0 rules begin with `required`; all ten slot 1–5 rules begin with `nullable`.

---

#### TC-A-05: 6-Slot Expansion — New Variables Present

**Covers:** AC-A-23, AC-A-24, stories.md TC-05

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available; pre-change variable count known (baseline).

**Steps:**
1. Parse the JSON and collect all `env_variable` values from the `variables` array.
2. Assert the following six values are all present: `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`.
3. Assert the total variable count equals the pre-change count plus 6 (no existing variables removed).

**Pass criteria:** All six new `env_variable` names present; total count = baseline + 6.

---

#### TC-A-06: New Slot Field Values

**Covers:** AC-A-25 to AC-A-28, stories.md TC-06

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. For each of `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`:
   a. Assert `default_value` is `""` (empty string).
   b. Assert `user_viewable` is `true`.
   c. Assert `user_editable` is `true`.
   d. Assert `field_type` is `"text"`.
   e. Assert `rules` begins with `"nullable"` (cross-check with TC-A-04).
2. Confirm no new slot has a non-empty `default_value`.

**Pass criteria:** All 6 new variables have empty default, viewable, editable, text field type, and nullable rules.

---

#### TC-A-07: Invalid GAMETYPE Warning in Descriptions (Static)

**Covers:** AC-A-29 (static portion)

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Steps:**
1. For each of `GAMETYPE_0` through `GAMETYPE_5`, extract the `description` field.
2. Assert each description contains warning text indicating that an invalid gametype value will cause the server to exit with an error at startup.
   - Acceptable forms include: "invalid", "exit", "error", "startup" (or semantically equivalent phrasing).
3. The warning must appear in **all six** GAMETYPE slot descriptions, not only GAMETYPE_0.

**Pass criteria:** All six GAMETYPE descriptions include a startup-exit warning for invalid values.

---

#### TC-A-08: JSON Validity

**Covers:** AC-A-30, stories.md TC-08

**Precondition:** Modified `egg-rainbow-six-3-raven-shield.json` available.

**Command:**
```
python -m json.tool egg-rainbow-six-3-raven-shield.json > /dev/null
```

**Steps:**
1. Run the command above from the `rainbow_six_3_raven_shield/` directory.
2. Assert exit code is `0`.
3. Assert no output to stderr.

**Pass criteria:** Exit code 0; no parse errors.

---

#### TC-A-09: PTDL_v2 Meta Version

**Covers:** AC-A-31

**Steps:**
1. Parse the JSON and read `meta.version`.
2. Assert exact value equals `"PTDL_v2"`.

**Pass criteria:** `meta.version` equals `"PTDL_v2"`.

---

#### TC-A-10: docker_images Key Equals Value

**Covers:** AC-A-32

**Steps:**
1. Parse the JSON and read the `docker_images` object.
2. Assert the object contains exactly one key.
3. Assert the key is `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`.
4. Assert the value equals the key (i.e., also `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`).

**Pass criteria:** Exactly one entry; key equals value equals full URI.

---

#### TC-A-11: Unmodified Variables Regression Check

**Covers:** AC-A-33, stories.md TC-09

**Precondition:** Pre-change egg JSON available as a baseline snapshot.

**Variables to check (must be byte-for-byte identical):**
`GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`

**Steps:**
1. For each variable in the list above, compare all five fields between baseline and modified file:
   - `name`, `description`, `default_value`, `rules`, `field_type`
2. Assert no field differs from the baseline value.
3. Assert `MAP_0` and `GAMETYPE_0` field values other than `description` (which is separately tested for no-regression in TC-A-01 and TC-A-02) are unchanged.

**Pass criteria:** Zero field-value differences on any unmodified variable.

---

### Track A — Empirical (Docker, where available)

---

#### TC-A-12: Invalid GAMETYPE Causes Non-Zero Exit (Empirical)

**Covers:** AC-A-29 (empirical portion), stories.md TC-07

**Precondition:** Docker available locally; image pullable.

**Command:**
```
docker run --rm \
  -e GAMETYPE_0=R6Game.InvalidType \
  ghcr.io/danpowell88/ravenshield_dedicatedserver
```

**Steps:**
1. Run the command above.
2. Capture container stdout and stderr.
3. Assert the container exits with a **non-zero** exit code.
4. Assert the output contains an error message from the `validate_gametype` function before any Wine process invocation output appears.
5. Assert no Wine startup lines (e.g., lines beginning with `wine:` or `wineserver`) appear before the error message.

**Pass criteria:** Non-zero exit; error message from `validate_gametype` visible; no Wine lines precede the error.

**Note:** If Docker is not available locally, this test case is deferred to the CI/CD pipeline or noted as a known gap. The static description warning check (TC-A-07) remains mandatory in all cases.

---

### Track B1 — Static (if Gate 1 = override variable confirmed)

---

#### TC-B1-01: New Env Var Object Fields (Static)

**Covers:** AC-B1-01, AC-B1-02

**Steps:**
1. Locate the new variable object in the `variables` array (env var name as confirmed by Gate 1, e.g., `GAMEFILES_DIR`).
2. Assert `default_value` is `"/home/container/gamefiles"` (or the maintainer-confirmed path).
3. Assert `user_viewable` is `true`.
4. Assert `user_editable` is `true`.
5. Assert `field_type` is `"text"`.
6. Assert `rules` contains `"required"`, `"string"`, and `"max:256"` (or a stricter rule if confirmed).

**Pass criteria:** All field values match spec.

---

#### TC-B1-02: New Env Var Description Content (Static)

**Covers:** AC-B1-03

**Steps:**
1. Extract the `description` field of the new variable.
2. Assert the description explains that the variable redirects game files from `/rvs` to a Pterodactyl-compatible path under `/home/container`.
3. Assert the description warns that changing the value to a non-writable path may break the server.

**Pass criteria:** Both explanatory and warning content present in description.

---

#### TC-B1-03: No Permission Warning in Egg Description (Static)

**Covers:** AC-B1-05

**Steps:**
1. Extract the top-level `description` field of the egg JSON.
2. Assert the field does **not** contain warning language about `/rvs` permission errors or the startup crash (i.e., the fix is in place, warning is unnecessary).

**Pass criteria:** No crash/permission warning present in egg description.

---

#### TC-B1-04: Container Starts Without Crash (Empirical)

**Covers:** AC-B1-04, stories.md TC-10

**Precondition:** Docker available; Gate 1 = override variable confirmed; override variable name known.

**Command (example with GAMEFILES_DIR):**
```
docker run --rm --user 1000:1000 \
  -e GAMEFILES_DIR=/home/container/gamefiles \
  ghcr.io/danpowell88/ravenshield_dedicatedserver
```

**Steps:**
1. Run the command above.
2. Capture stdout output for at least 30 seconds or until the container exits.
3. Assert the container does **not** exit immediately (within the first 5 seconds) with exit code 1 and "Permission denied" in output.
4. Assert at least one line of entrypoint output is visible before any failure.
5. The container reaching `"OpenRVS is up to date"` in stdout is the ideal success state and constitutes a full pass; a partial pass is accepted if startup progresses past the `/rvs` mkdir step without a permission error.

**Pass criteria:** No immediate "Permission denied" exit; at least one entrypoint output line visible.

---

### Track B2 — Static (if Gate 1 = no override exists)

---

#### TC-B2-01: Crash Warning Present and Correctly Positioned in Egg Description

**Covers:** AC-B2-01, AC-B2-02, stories.md TC-11

**Steps:**
1. Extract the top-level `description` field of the egg JSON.
2. Assert the field contains all of the following content:
   a. A statement that the container will crash immediately on startup with "Permission denied".
   b. An explanation that `/rvs` is owned by root and the container runs as uid 1000 (or equivalent phrasing).
   c. A statement that this is an image-layer defect not resolvable by egg JSON edits alone.
   d. The Wings bind-mount workaround (mounting the Pterodactyl data volume at `/rvs`).
3. Assert the warning content appears **before** any standard server description prose (i.e., it is at the beginning of the `description` value, not appended after existing text).

**Pass criteria:** All four warning content elements present; warning appears before other description content.

---

#### TC-B2-02: Egg Description Does Not Claim Crash Is Fixed

**Covers:** AC-B2-03

**Steps:**
1. Extract the top-level `description` field.
2. Assert the field does **not** contain any claim that the startup crash has been resolved or fixed.

**Pass criteria:** No claim of crash resolution present.

---

#### TC-B2-03: README Known Issue Section

**Covers:** AC-B2-04

**Precondition:** `rainbow_six_3_raven_shield/README.md` exists.

**Steps:**
1. Open `rainbow_six_3_raven_shield/README.md`.
2. Assert a section heading exists for "Known Issue: Startup Crash" (or semantically equivalent heading).
3. Under that section, assert all of the following are present:
   a. Statement that the container crashes on startup with exit code 1 and "Permission denied" output.
   b. Root cause explanation: Docker image hardcodes `/rvs`; Pterodactyl mounts at `/home/container`; uid 1000 cannot write to `/rvs`.
   c. Step-by-step Wings bind-mount workaround.

**Pass criteria:** Section heading present; all three content elements (a), (b), (c) confirmed.

---

## 4. Validation Matrix

| Story AC | Description (abbreviated) | Test Case(s) | Method |
|---|---|---|---|
| AC-A-01 to AC-A-05 | MAP_1–MAP_5 descriptions contain full inline map list | TC-A-01 | Static |
| AC-A-06 | MAP_0 description unchanged | TC-A-01 | Static (regression) |
| AC-A-07 to AC-A-11 | GAMETYPE_1–GAMETYPE_5 descriptions contain all six gametypes with labels | TC-A-02 | Static |
| AC-A-12 | GAMETYPE_0 description unchanged | TC-A-02 | Static (regression) |
| AC-A-13 to AC-A-18 | All GAMETYPE rules contain max:128 | TC-A-03 | Static |
| AC-A-19 | MAP_0 rules begins with required | TC-A-04 | Static |
| AC-A-20 | GAMETYPE_0 rules begins with required | TC-A-04 | Static |
| AC-A-21 | MAP_1–MAP_5 rules begin with nullable | TC-A-04 | Static |
| AC-A-22 | GAMETYPE_1–GAMETYPE_5 rules begin with nullable | TC-A-04 | Static |
| AC-A-23 | MAP_3, MAP_4, MAP_5 present in variables array | TC-A-05 | Static |
| AC-A-24 | GAMETYPE_3, GAMETYPE_4, GAMETYPE_5 present | TC-A-05 | Static |
| AC-A-25 | New map slots have empty default and nullable rules | TC-A-06 | Static |
| AC-A-26 | New gametype slots have empty default and nullable rules | TC-A-06 | Static |
| AC-A-27 | New slots have user_viewable and user_editable true | TC-A-06 | Static |
| AC-A-28 | New slots have field_type text | TC-A-06 | Static |
| AC-A-29 | GAMETYPE descriptions warn invalid value causes exit; container exits non-zero on invalid value | TC-A-07, TC-A-12 | Static + Empirical |
| AC-A-30 | JSON valid (python -m json.tool exits 0) | TC-A-08 | Static |
| AC-A-31 | meta.version is PTDL_v2 | TC-A-09 | Static |
| AC-A-32 | docker_images key equals value equals full URI | TC-A-10 | Static |
| AC-A-33 | Unmodified variables byte-for-byte identical | TC-A-11 | Static (regression) |
| AC-B1-01 | GAMEFILES_DIR variable present with correct default | TC-B1-01 | Static (B1 only) |
| AC-B1-02 | New variable has correct field_type, rules, viewable/editable | TC-B1-01 | Static (B1 only) |
| AC-B1-03 | New variable description explains purpose and warns | TC-B1-02 | Static (B1 only) |
| AC-B1-04 | Container starts without Permission denied crash | TC-B1-04 | Empirical (B1 only) |
| AC-B1-05 | Egg description contains no crash warning | TC-B1-03 | Static (B1 only) |
| AC-B2-01 | Egg description contains crash warning with all four elements | TC-B2-01 | Static (B2 only) |
| AC-B2-02 | Warning appears at top of description | TC-B2-01 | Static (B2 only) |
| AC-B2-03 | Description does not claim crash is fixed | TC-B2-02 | Static (B2 only) |
| AC-B2-04 | README has Known Issue section with cause and workaround | TC-B2-03 | Static (B2 only) |

---

## 5. Limitations and Dependencies

### 5.1 Track B Gate Dependency

Track B test cases are **gated on Gate 1 outcome**. The Gate 1 question (does the Docker image expose an env var to override `/rvs`?) must be resolved by the PO before Track B tests can be selected. Exactly one of B1 or B2 applies — the QA engineer must confirm which sub-track is active before executing Track B tests.

### 5.2 Docker Availability

TC-A-12 (invalid GAMETYPE exit code) and TC-B1-04 (crash resolution) require Docker locally with access to `ghcr.io/danpowell88/ravenshield_dedicatedserver`. If Docker is not available:
- TC-A-12 is deferred; static description check TC-A-07 still runs and is mandatory.
- TC-B1-04 is deferred; all static B1 tests still run.

Document any deferred Docker tests explicitly in the QA sign-off report.

### 5.3 Baseline Snapshot

TC-A-11 (regression check) requires a snapshot of the pre-change egg JSON. If no baseline is committed in the repository, the developer must supply the unmodified original file before QA begins.

### 5.4 External Dependencies (Out of Scope)

- Archive.org game files URL liveness is a runtime dependency; not testable statically.
- Live Pterodactyl panel import is not required for this BUG_FIX pipeline; all Track A validation is achievable by static file inspection.

---

## 6. Definition of Done — QA Sign-Off Checklist

**Track A (all required):**
- [ ] TC-A-01: MAP slot descriptions verified (all 18 map names inline, no cross-references)
- [ ] TC-A-02: GAMETYPE slot descriptions verified (all 6 gametypes with labels, no cross-references)
- [ ] TC-A-03: All GAMETYPE rules contain max:128, none contain max:64
- [ ] TC-A-04: MAP_0 and GAMETYPE_0 rules begin with required; slots 1–5 begin with nullable
- [ ] TC-A-05: All 6 new env_variable names present in variables array; count = baseline + 6
- [ ] TC-A-06: New slots have empty default, user_viewable true, user_editable true, field_type text
- [ ] TC-A-07: All six GAMETYPE descriptions contain invalid-value startup-exit warning
- [ ] TC-A-08: JSON validity confirmed (python -m json.tool exits 0)
- [ ] TC-A-09: meta.version is "PTDL_v2"
- [ ] TC-A-10: docker_images key equals value equals full URI
- [ ] TC-A-11: All unmodified variables byte-for-byte identical to baseline
- [ ] TC-A-12: Invalid GAMETYPE causes non-zero exit (or deferred with documented reason)

**Track B (exactly one sub-track — check the active one):**
- [ ] **B1 active:** TC-B1-01, TC-B1-02, TC-B1-03 pass (static); TC-B1-04 pass or deferred with documented reason
- [ ] **B2 active:** TC-B2-01, TC-B2-02, TC-B2-03 pass (static)

---

*End of Test Strategy*
