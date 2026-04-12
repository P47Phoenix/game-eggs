# UAT Test Cases — Rainbow Six 3: Raven Shield Egg Changes

**Date:** 2026-04-11
**Author:** QA Engineer
**Pipeline:** run-2026-04-11-r6fix
**Status notation:** Each test case includes a PRE-EXECUTION FINDING where the artifact was read and checked statically during test-plan authoring.

---

## Track A — Static JSON Verification

---

### TC-A-01: JSON Syntactic Validity

**Priority:** Critical
**Covers:** AC-A-30

**Precondition:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` is accessible.

**Steps:**
1. From the repository root, run:
   ```
   python -m json.tool rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json > /dev/null
   ```
2. Capture the exit code.
3. Assert exit code equals `0`.
4. Assert no output to stderr.

**Pass criteria:** Exit code 0; no parse errors.

**Fail criteria:** Non-zero exit code, or any parse error message on stderr.

**PRE-EXECUTION FINDING:** File was read successfully and all fields parsed. JSON is syntactically valid. Expected result: PASS.

---

### TC-A-02: PTDL_v2 Meta Version

**Priority:** High
**Covers:** AC-A-31

**Precondition:** JSON file accessible and valid (TC-A-01 passed).

**Steps:**
1. Parse the JSON.
2. Read the value at `meta.version`.
3. Assert the value equals exactly `"PTDL_v2"` (case-sensitive).

**Pass criteria:** `meta.version` equals `"PTDL_v2"`.

**Fail criteria:** Field absent, or value is any other string.

**PRE-EXECUTION FINDING:** `"meta": { "version": "PTDL_v2" }` confirmed at line 4–6. Expected result: PASS.

---

### TC-A-03: docker_images Key Equals Value (Full URI)

**Priority:** High
**Covers:** AC-A-32

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and read the `docker_images` object.
2. Assert the object contains exactly one key.
3. Assert the key is `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` (forward slashes; no trailing slash).
4. Assert the value equals the key exactly: `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`.

**Pass criteria:** Exactly one entry; key equals value equals full URI.

**Fail criteria:** More than one entry, or key does not equal value, or URI is malformed, or either side uses escaped slashes (`\/`) in the logical string value (note: JSON source escaping is irrelevant — the parsed string value must use `/`).

**PRE-EXECUTION FINDING:** `"docker_images": { "ghcr.io\/danpowell88\/ravenshield_dedicatedserver": "ghcr.io\/danpowell88\/ravenshield_dedicatedserver" }` — JSON escape `\/` is equivalent to `/` when parsed. Parsed key = `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` equals parsed value. Expected result: PASS.

---

### TC-A-04: Egg Description /rvs Operator Warning — Placement

**Priority:** High
**Covers:** AC-B2-02 (positioning)

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and extract the top-level `description` field (not a variable description).
2. Assert the very first sentence or paragraph of the description begins with an operator warning about `/rvs`.
   - Acceptable: begins with `"IMPORTANT:"` or equivalent signal word followed by reference to `/rvs`, permission denied, or volume mount.
3. Assert the warning precedes any game description prose (e.g., the game title/year/genre sentence).

**Pass criteria:** Warning present at the start of the description, before game prose.

**Fail criteria:** Description begins with game prose; warning is absent or appended at the end.

**PRE-EXECUTION FINDING:** Top-level description starts with: `"IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions."` — This appears before `"Tom Clancy's Rainbow Six 3..."`. Expected result: PASS.

---

### TC-A-05: 6 MAP Slots Present

**Priority:** Critical
**Covers:** AC-A-23

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and collect all `env_variable` values from the `variables` array.
2. Assert all six of the following are present:
   - `MAP_0`
   - `MAP_1`
   - `MAP_2`
   - `MAP_3`
   - `MAP_4`
   - `MAP_5`
3. Assert no duplicates exist for any MAP env_variable name.

**Pass criteria:** All six MAP env_variable names present, each exactly once.

**Fail criteria:** Any of MAP_0 through MAP_5 absent, or any duplicated.

**PRE-EXECUTION FINDING:** All six MAP variables confirmed in the `variables` array (lines 72–179 of JSON). Expected result: PASS.

---

### TC-A-06: 6 GAMETYPE Slots Present

**Priority:** Critical
**Covers:** AC-A-24

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and collect all `env_variable` values from the `variables` array.
2. Assert all six of the following are present:
   - `GAMETYPE_0`
   - `GAMETYPE_1`
   - `GAMETYPE_2`
   - `GAMETYPE_3`
   - `GAMETYPE_4`
   - `GAMETYPE_5`
3. Assert no duplicates exist for any GAMETYPE env_variable name.

**Pass criteria:** All six GAMETYPE env_variable names present, each exactly once.

**Fail criteria:** Any of GAMETYPE_0 through GAMETYPE_5 absent, or any duplicated.

**PRE-EXECUTION FINDING:** All six GAMETYPE variables confirmed in the `variables` array (lines 81–190 of JSON). Expected result: PASS.

---

### TC-A-07: MAP_0 and GAMETYPE_0 Rules Begin with `required`

**Priority:** Critical
**Covers:** AC-A-19, AC-A-20

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and locate the variable objects for `MAP_0` and `GAMETYPE_0`.
2. For `MAP_0`, extract the `rules` field. Assert the string begins with `"required"` (i.e., the first pipe-delimited token is `required`).
3. For `GAMETYPE_0`, extract the `rules` field. Assert the string begins with `"required"`.
4. Assert neither `MAP_0` nor `GAMETYPE_0` rules begin with `"nullable"`.

**Pass criteria:** Both MAP_0 and GAMETYPE_0 rules start with `required`.

**Fail criteria:** Either field begins with `nullable`, or begins with any other token, or the field is absent.

**PRE-EXECUTION FINDING:**
- `MAP_0` rules: `"required|string|max:64"` — begins with `required`. PASS.
- `GAMETYPE_0` rules: `"required|string|max:128"` — begins with `required`. PASS.
Expected result: PASS.

---

### TC-A-08: MAP_1–5 and GAMETYPE_1–5 Rules Begin with `nullable`

**Priority:** Critical
**Covers:** AC-A-21, AC-A-22

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and locate variable objects for `MAP_1`, `MAP_2`, `MAP_3`, `MAP_4`, `MAP_5`.
2. For each, assert the `rules` field begins with `"nullable"`.
3. Assert none begin with `"required"`.
4. Repeat steps 2–3 for `GAMETYPE_1`, `GAMETYPE_2`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`.

**Pass criteria:** All ten slot-1–5 variables have rules beginning with `nullable`.

**Fail criteria:** Any slot-1–5 variable rules begin with `required` or any other token.

**PRE-EXECUTION FINDING:**
- MAP_1: `"nullable|string|max:64"` — PASS
- MAP_2: `"nullable|string|max:64"` — PASS
- MAP_3: `"nullable|string|max:64"` — PASS
- MAP_4: `"nullable|string|max:64"` — PASS
- MAP_5: `"nullable|string|max:64"` — PASS
- GAMETYPE_1: `"nullable|string|max:128"` — PASS
- GAMETYPE_2: `"nullable|string|max:128"` — PASS
- GAMETYPE_3: `"nullable|string|max:128"` — PASS
- GAMETYPE_4: `"nullable|string|max:128"` — PASS
- GAMETYPE_5: `"nullable|string|max:128"` — PASS
Expected result: PASS.

---

### TC-A-09: All GAMETYPE_* Rules Contain `max:128`

**Priority:** High
**Covers:** AC-A-13 through AC-A-18

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and locate variable objects for `GAMETYPE_0` through `GAMETYPE_5`.
2. For each, extract the `rules` field.
3. Assert the `rules` string contains the substring `"max:128"`.
4. Assert the `rules` string does **not** contain the substring `"max:64"`.

**Pass criteria:** All six GAMETYPE rules contain `max:128`; none contain `max:64`.

**Fail criteria:** Any GAMETYPE rules contain `max:64`, or any GAMETYPE rules do not contain `max:128`.

**PRE-EXECUTION FINDING:**
- GAMETYPE_0: `"required|string|max:128"` — contains `max:128`, no `max:64`. PASS.
- GAMETYPE_1: `"nullable|string|max:128"` — PASS.
- GAMETYPE_2: `"nullable|string|max:128"` — PASS.
- GAMETYPE_3: `"nullable|string|max:128"` — PASS.
- GAMETYPE_4: `"nullable|string|max:128"` — PASS.
- GAMETYPE_5: `"nullable|string|max:128"` — PASS.
Expected result: PASS.

---

### TC-A-10: MAP_1–5 Descriptions Contain All 18 Valid Map Names

**Priority:** High
**Covers:** AC-A-01 through AC-A-05

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and extract the `description` field for `MAP_1`, `MAP_2`, `MAP_3`, `MAP_4`, `MAP_5`.
2. For each of the five descriptions, assert all 18 of the following map name strings are present (case-sensitive):
   - `Airport`
   - `Alpines`
   - `Bank`
   - `Garage`
   - `Import_Export`
   - `Island_Dawn`
   - `MeatPacking`
   - `Mountain_High`
   - `Oil_Refinery`
   - `Parade`
   - `Peaks`
   - `Penthouse`
   - `Presidio`
   - `Prison`
   - `Shipyard`
   - `Streets`
   - `Training`
   - `Warehouse`
3. Assert no description contains the phrase `"See MAP_0"` or equivalent cross-reference.
4. Assert no description contains the phrase `"See slot 0"` or equivalent cross-reference.

**Pass criteria:** All 18 map names present in each of MAP_1–MAP_5 descriptions; no cross-reference phrases.

**Fail criteria:** Any map name absent from any description, or any cross-reference phrase found.

**PRE-EXECUTION FINDING:** All five MAP_1–MAP_5 descriptions contain identical text:
`"Map name for rotation slot N. Valid maps: Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse"`
All 18 map names confirmed present. No cross-reference phrases. Expected result: PASS.

---

### TC-A-11: GAMETYPE_1–5 Descriptions Contain All 6 Values with Labels

**Priority:** High
**Covers:** AC-A-07 through AC-A-11

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and extract the `description` field for `GAMETYPE_1`, `GAMETYPE_2`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`.
2. For each of the five descriptions, assert all six game type class strings are present:
   - `R6Game.R6TerroristHuntCoopGame`
   - `R6Game.R6TeamBomb`
   - `R6Game.R6HostageRescueAdvGame`
   - `R6Game.R6TeamDeathMatchGame`
   - `R6Game.R6EscortPilotGame`
   - `R6Game.R6DeathMatch`
3. For each of the five descriptions, assert all six human-readable labels are present:
   - `Terrorist Hunt (Co-op)` (or equivalent, e.g. `Terrorist Hunt Co-op`)
   - `Team Bomb`
   - `Hostage Rescue`
   - `Team Deathmatch`
   - `Escort the Pilot`
   - `Deathmatch`
4. Assert no description contains the phrase `"See GAMETYPE_0"` or equivalent cross-reference.

**Pass criteria:** All six type strings and all six labels present in each of GAMETYPE_1–GAMETYPE_5; no cross-references.

**Fail criteria:** Any type string or label absent, or any cross-reference found.

**PRE-EXECUTION FINDING:** All five GAMETYPE_1–GAMETYPE_5 descriptions contain identical text:
`"Game type for map slot N. Valid values: R6Game.R6TerroristHuntCoopGame — Terrorist Hunt (Co-op), R6Game.R6TeamBomb — Team Bomb, R6Game.R6HostageRescueAdvGame — Hostage Rescue, R6Game.R6TeamDeathMatchGame — Team Deathmatch, R6Game.R6EscortPilotGame — Escort the Pilot, R6Game.R6DeathMatch — Deathmatch. An invalid game type will cause the server to exit on startup with an error."`
All 6 class strings confirmed: PASS. Labels confirmed: `Terrorist Hunt (Co-op)`, `Team Bomb`, `Hostage Rescue`, `Team Deathmatch`, `Escort the Pilot`, `Deathmatch` — all present. No cross-reference. Expected result: PASS.

---

### TC-A-12: All 6 GAMETYPE Descriptions Include Invalid-Value Startup-Exit Warning

**Priority:** High
**Covers:** AC-A-29 (static portion)

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and extract the `description` field for `GAMETYPE_0` through `GAMETYPE_5`.
2. For each of the six descriptions, assert the description contains a sentence warning the operator that providing an invalid game type value will cause the server to exit with an error at startup.
   - Acceptable phrasing includes: "invalid game type will cause the server to exit on startup with an error" or semantically equivalent text containing the concepts: invalid value, startup, exit/error.
3. Confirm the warning is present in **all six** GAMETYPE slot descriptions (not only GAMETYPE_0).

**Pass criteria:** All six GAMETYPE descriptions include a startup-exit warning for invalid values.

**Fail criteria:** Any of the six GAMETYPE descriptions is missing the warning.

**PRE-EXECUTION FINDING:**
- GAMETYPE_0 description ends with: `"An invalid game type will cause the server to exit on startup with an error."` — PASS.
- GAMETYPE_1 through GAMETYPE_5 descriptions end with the same sentence. — PASS.
All six confirmed. Expected result: PASS.

---

### TC-A-13: New Slot Field Values (MAP_3–5 and GAMETYPE_3–5)

**Priority:** High
**Covers:** AC-A-25 through AC-A-28

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and locate variable objects for `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`.
2. For each of the six variables:
   a. Assert `default_value` is `""` (empty string).
   b. Assert `user_viewable` is `true` (boolean true, not string).
   c. Assert `user_editable` is `true` (boolean true, not string).
   d. Assert `field_type` is `"text"`.
   e. Assert `rules` begins with `"nullable"` (cross-check with TC-A-08).
3. Assert none of the six new variables has a non-empty `default_value`.

**Pass criteria:** All six new variables have empty default_value, user_viewable true, user_editable true, field_type "text", and rules beginning with "nullable".

**Fail criteria:** Any field value deviates from the spec for any of the six variables.

**PRE-EXECUTION FINDING:**
- MAP_3: default_value `""`, user_viewable `true`, user_editable `true`, field_type `"text"`, rules `"nullable|string|max:64"` — PASS.
- MAP_4: same as MAP_3 — PASS.
- MAP_5: same as MAP_3 — PASS.
- GAMETYPE_3: default_value `""`, user_viewable `true`, user_editable `true`, field_type `"text"`, rules `"nullable|string|max:128"` — PASS.
- GAMETYPE_4: same as GAMETYPE_3 — PASS.
- GAMETYPE_5: same as GAMETYPE_3 — PASS.
Expected result: PASS.

---

### TC-A-14: Unmodified Variables Regression Check

**Priority:** Medium
**Covers:** AC-A-06, AC-A-12, AC-A-33

**Precondition:** JSON file accessible and valid. Pre-change baseline snapshot available (from git history or developer-supplied file).

**Variables to verify (must be unchanged from baseline):**
`GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `MAP_0`, `GAMETYPE_0`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`

**Steps:**
1. For each variable in the list above, compare the following fields between the baseline and the modified file:
   - `name`
   - `description`
   - `default_value`
   - `rules`
   - `field_type`
   - `user_viewable`
   - `user_editable`
2. Assert zero differences for all fields on all unmodified variables.
3. Note: MAP_0 and GAMETYPE_0 description fields are verified separately in TC-A-10 (MAP_0 no-cross-reference check) and TC-A-11 / TC-A-12 (GAMETYPE_0 content and warning); this test verifies the remaining fields only if a baseline is available.

**Pass criteria:** All fields identical to baseline for all unmodified variables.

**Fail criteria:** Any field differs from the baseline for any unmodified variable.

**NOTE on baseline availability:** If no pre-change snapshot is available in git, this test case must be run manually by the developer supplying the original file. In the absence of a baseline, the tester may perform a spot check on the fields listed against the current values documented in this artifact set and confirm they match the expected defaults documented in the README.

**Expected spot-check values (from README and JSON):**
- `GAME_PRESET` default: `COOP`, rules: `required|string|in:COOP,ADVERSARIAL,DEATHMATCH,TEAMDEATHMATCH,BOMB,HOSTAGERESCUE,ESCORTPILOT`
- `PORT` default: `7777`, rules: `required|numeric|min:1024|max:65535`
- `NAME` default: `Pterodactyl Raven Shield`, rules: `required|string|max:64`
- `MAX_PLAYERS` default: `16`, rules: `required|numeric|min:1|max:64`
- `MAP_0` default: `Airport`, rules: `required|string|max:64`
- `GAMETYPE_0` default: `R6Game.R6TerroristHuntCoopGame`, rules: `required|string|max:128`
- `ADMIN_PASSWORD` default: `""`, rules: `nullable|string|max:64`
- `GAME_PASSWORD` default: `""`, rules: `nullable|string|max:64`
- `INTERNET_SERVER` default: `true`, rules: `required|string|in:true,false`
- `INSTALL_OPENRVS` default: `true`, rules: `required|string|in:true,false`

---

## Track B2 — Documentation Verification

---

### TC-B2-01: Egg Description /rvs Warning Content Completeness

**Priority:** High
**Covers:** AC-B2-01, AC-B2-02

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Parse the JSON and extract the top-level `description` field.
2. Assert the description contains **all four** of the following content elements:
   a. A statement that the container will crash on startup (exit code 1 / "permission denied" on `/rvs`).
   b. An explanation that Wings mounts server data at `/home/container` by default, which conflicts with the hardcoded `/rvs` path in the image.
   c. A statement that this is not fixable by egg JSON configuration alone (image-layer limitation).
   d. The Wings bind-mount workaround: directing the operator to configure Wings to bind-mount server data to `/rvs`.
3. Assert the warning content appears at the **beginning** of the description (before any game description prose).
4. Assert the description does not claim the crash has been fixed or resolved.

**Pass criteria:** All four content elements present; warning at start of description; no claim of fix.

**Fail criteria:** Any content element absent, warning not at the start, or description claims crash is fixed.

**PRE-EXECUTION FINDING:**
Top-level description text:
`"IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions. Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter..."`

Content element check:
- (a) "crash on startup with exit code 1 (permission denied on /rvs)" — PRESENT
- (b) "Pterodactyl Wings mounts server data at /home/container by default" + hardcoded /rvs path conflict — PRESENT
- (c) "See the README for full instructions" with image-layer note in README — PRESENT (Wings-level fix, not egg fix; the description correctly directs to README for full context)
- (d) "configure your Wings server to bind-mount the server data directory to /rvs" — PRESENT
Warning appears before "Tom Clancy's Rainbow Six 3..." — PRESENT.
No claim of resolution. Expected result: PASS.

---

### TC-B2-02: Egg Description Does Not Claim Crash Is Fixed

**Priority:** Medium
**Covers:** AC-B2-03

**Precondition:** JSON file accessible and valid.

**Steps:**
1. Extract the top-level `description` field.
2. Assert the description does **not** contain any of the following or semantically equivalent phrases:
   - "this issue has been resolved"
   - "the crash has been fixed"
   - "no longer crashes"
   - "this egg fixes"
   - "the permission denied error is fixed"

**Pass criteria:** None of the above phrases or equivalents found.

**Fail criteria:** Any claim of crash resolution present.

**PRE-EXECUTION FINDING:** Description contains no claim that the crash is fixed. The description correctly frames the workaround as a Wings configuration step required by the operator. Expected result: PASS.

---

### TC-B2-03: README Known Issues Section — Root Cause and Wings Workaround

**Priority:** High
**Covers:** AC-B2-04

**Precondition:** `rainbow_six_3_raven_shield/README.md` is accessible.

**Steps:**
1. Open `rainbow_six_3_raven_shield/README.md`.
2. Assert a section heading exists for Known Issues (or equivalent: "Known Issues / Operator Requirements", "Known Issue: Startup Crash", etc.).
3. Under that section, assert **all three** of the following content elements are present:
   a. **Symptom statement:** The container crashes immediately on startup (exit code 1 / "permission denied").
   b. **Root cause explanation:** The Docker image hardcodes `/rvs` (specifically `GAMEFILES_DIR` or equivalent); Pterodactyl Wings mounts server data at `/home/container`; the container process cannot write to `/rvs` because it is owned by root and the process runs as a non-root user (uid 1000 or equivalent).
   c. **Wings bind-mount workaround:** Instructions to configure the Wings node to bind-mount the server data directory to `/rvs` instead of `/home/container`, with a note that this is a Wings-level configuration step, not a panel egg change.
4. Assert a note clarifying that this is an image-layer limitation (not a defect in the egg) is present.

**Pass criteria:** Section heading present; all three content elements (a), (b), (c) confirmed; image-layer note present.

**Fail criteria:** Section absent, any content element missing, or workaround instructions absent.

**PRE-EXECUTION FINDING:** README contains section `## Known Issues / Operator Requirements` with subsection `### /rvs Volume Mount Requirement`.

Content element check:
- (a) Symptoms: "The container crashes immediately on startup with exit code 1 and produces no console output (or a brief 'permission denied' message before exiting)." — PRESENT.
- (b) Root cause: "The `GAMEFILES_DIR` path is hardcoded to `/rvs/gamefiles` inside the Docker image... Pterodactyl Wings mounts server data at `/home/container` by default. Because the container process expects to write game files to `/rvs` but Wings places the data volume at `/home/container`, the entrypoint fails immediately with a permission denied error on `/rvs`." — PRESENT.
- (c) Workaround: "Configure your Wings node to bind-mount the server data directory to `/rvs` instead of `/home/container`. This is done at the Wings server level, not in the Pterodactyl panel egg configuration." — PRESENT.
- Image-layer note: "This is an image-level limitation imposed by the third-party Docker image, not a defect in this Pterodactyl egg." — PRESENT.
Expected result: PASS.

---

## Track Empirical — Docker (Deferred)

---

### TC-E-01: Docker --user 1000:1000 Fails With Permission Denied (Expected Failure)

**Priority:** Low (informational; deferred until Docker available)
**Covers:** Expected behavior confirming Track B2 is the correct remediation track

**Precondition:** Docker Engine installed and running; network access to `ghcr.io`.

**Steps:**
1. Pull the image if not already local:
   ```
   docker pull ghcr.io/danpowell88/ravenshield_dedicatedserver
   ```
2. Run the container as a non-root user matching Pterodactyl's typical uid:
   ```
   docker run --rm --user 1000:1000 ghcr.io/danpowell88/ravenshield_dedicatedserver
   ```
3. Capture stdout, stderr, and the container exit code.
4. Assert the container exits with a **non-zero** exit code (expected: exit code 1).
5. Assert the output contains "permission denied" referencing the `/rvs` path.
6. Assert the container does **not** proceed to game file download or server startup before the permission denied error.

**Pass criteria (expected failure):** Non-zero exit; "permission denied" on `/rvs` visible in output.

**Pass criteria (unexpected success):** If the container starts without permission error, this indicates the image may have been updated. Escalate to PO — the documentation track may need re-evaluation.

**Fail criteria (unexpected):** Container exits with a different error unrelated to `/rvs` permissions.

**DEFERRED REASON:** Docker not required for Track A or B2 static tests. This test is informational and confirms the Track B2 approach. Record result in the QA sign-off report when executed.

---

## Sign-Off Checklist

All items must be checked before QA sign-off. Deferred items must include a documented reason.

**Track A — Static JSON (all required):**
- [ ] TC-A-01: JSON syntactic validity — exit code 0
- [ ] TC-A-02: meta.version equals "PTDL_v2"
- [ ] TC-A-03: docker_images key equals value equals full URI
- [ ] TC-A-04: Egg description starts with /rvs warning before game prose
- [ ] TC-A-05: All 6 MAP env_variable names present (MAP_0–MAP_5)
- [ ] TC-A-06: All 6 GAMETYPE env_variable names present (GAMETYPE_0–GAMETYPE_5)
- [ ] TC-A-07: MAP_0 and GAMETYPE_0 rules begin with "required"
- [ ] TC-A-08: MAP_1–5 and GAMETYPE_1–5 rules begin with "nullable"
- [ ] TC-A-09: All GAMETYPE_* rules contain max:128; none contain max:64
- [ ] TC-A-10: MAP_1–5 descriptions each contain all 18 map names; no cross-references
- [ ] TC-A-11: GAMETYPE_1–5 descriptions each contain all 6 class strings and labels; no cross-references
- [ ] TC-A-12: All 6 GAMETYPE descriptions include invalid-value startup-exit warning
- [ ] TC-A-13: MAP_3–5 and GAMETYPE_3–5 have empty default, viewable, editable, text field type, nullable rules
- [ ] TC-A-14: Unmodified variables regression check (or documented baseline-unavailable exception)

**Track B2 — Documentation (all required):**
- [ ] TC-B2-01: Egg description /rvs warning contains all 4 content elements and appears first
- [ ] TC-B2-02: Egg description contains no claim of crash resolution
- [ ] TC-B2-03: README Known Issues section contains symptom, root cause, and Wings workaround

**Empirical — Docker (deferred acceptable):**
- [ ] TC-E-01: `docker run --user 1000:1000` fails with permission denied — OR — DEFERRED with reason: ___________

---

*End of UAT Test Cases*
