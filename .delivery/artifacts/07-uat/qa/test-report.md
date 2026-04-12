# UAT Test Report: Tom Clancy's Rainbow Six 3: Raven Shield Pterodactyl Egg

**Date:** 2026-04-11
**Tester:** QA Engineer
**Test Strategy Ref:** `.delivery/artifacts/05-plan/qa/test-strategy.md`
**Egg File:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**Game README:** `rainbow_six_3_raven_shield/README.md`
**Root README:** `README.md`

---

## Static Test Case Results

---

### TC-01-1: JSON Schema Validation

**Description:** Verify the egg JSON file parses without errors and all required top-level keys are present.

**Result: PASS**

- `python3 -m json.tool` exits 0 — no parse errors.
- All 13 required top-level keys present: `_comment`, `meta`, `exported_at`, `name`, `author`, `description`, `features`, `docker_images`, `file_denylist`, `startup`, `config`, `scripts`, `variables`.
- `meta` contains `version` and `update_url`.
- `config` contains `files`, `startup`, `logs`, and `stop`.
- `scripts.installation` contains `script`, `container`, and `entrypoint`.
- `variables` is a JSON array with exactly 14 elements.

---

### TC-01-3 (Static portion): Environment Variable Verification

**Description:** For each of the 14 variables, verify `env_variable`, `default_value`, `rules`, `field_type`, `user_viewable`, `user_editable`, and nullable semantics against the strategy table.

**Result: FAIL**

The following discrepancies were found:

#### FAIL-1: Variable ordering does not match the strategy table

The strategy (Section 2.1.3) specifies the variables in this order: `GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `MAP_0`, `MAP_1`, `MAP_2`, `GAMETYPE_0`, `GAMETYPE_1`, `GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`.

The actual JSON interleaves MAP and GAMETYPE slots: `MAP_0`, **`GAMETYPE_0`**, `MAP_1`, **`GAMETYPE_1`**, `MAP_2`, **`GAMETYPE_2``. This is a cosmetic ordering deviation; the logical grouping is arguably better in the JSON, but it does not match the specified order. **Severity: Low — panel import is unaffected; flag for spec alignment.**

#### FAIL-2: NAME default value is wrong

- Expected (strategy §2.1.3 row 3): `"Pterodactyl Raven Shield"`
- Actual: `"My RavenShield Server"`
- **Severity: Medium — the panel UI default shown to users does not match the specification.**

#### FAIL-3: MAX_PLAYERS default value is wrong

- Expected (strategy §2.1.3 row 4): `"16"`
- Actual: `"8"`
- **Severity: Medium — a new server will launch with half the specified default player cap.**

#### FAIL-4: MAP_0 rules are wrong

- Expected (strategy §2.1.3 row 5): `"required|string|max:64"`
- Actual: `"nullable|string|max:64"`
- MAP_0 is the mandatory first map slot; making it nullable allows the panel to accept an empty value that would cause a server startup failure.
- **Severity: High — incorrect nullability rule on a required field.**

#### FAIL-5: GAMETYPE_0 rules are wrong

- Expected (strategy §2.1.3 row 8): `"required|string|max:128"`
- Actual: `"nullable|string|max:64"` (also wrong `max` value: 64 vs 128)
- GAMETYPE_0 is the mandatory first game type slot; same issue as MAP_0 above. Additionally the `max` constraint is `64` instead of `128`, which would silently truncate the longest valid gametype strings (all six valid values are under 64 chars, so no practical impact, but it contradicts the spec).
- **Severity: High — incorrect nullability and max constraint on a required field.**

#### FAIL-6: ADMIN_PASSWORD and GAME_PASSWORD rules use max:32 instead of max:64

- Expected (strategy §2.1.3 rows 11 and 12): `"nullable|string|max:64"`
- Actual: `"nullable|string|max:32"`
- **Severity: Low — nullable and base type are correct; max length is half the specified limit.**

#### FAIL-7: INSTALL_OPENRVS has user_editable: false

- Expected (strategy §2.1.3): `user_editable: true` for all 14 variables.
- Actual: `user_editable: false` for `INSTALL_OPENRVS`.
- The strategy explicitly states "user_editable: true for all 14" and cross-references FR-13: "No variable has `user_editable: true` with `user_viewable: false`." While the FR-13 rule is not violated (the field is viewable but not editable), locking `INSTALL_OPENRVS` from user editing is a deliberate design choice that contradicts the stated spec requirement of full user editability.
- **Severity: Medium — panel users cannot toggle OpenRVS installation despite the spec requiring editability.**

#### Note on MAP_1 default (pre-existing discrepancy, strategy §6)

The strategy flags a conflict: stories.md says `MAP_1` default = `"Alpines"`, PRD FR-9 says `"Bank"`. The egg implements `"Alpines"` (matching stories.md). The README env var table also shows `"Alpines"`. No verdict can be rendered without implementer resolution; this is recorded as a known open question, not a new defect.

#### Note on MAP_2 default

- Expected (strategy §2.1.3 row 7): `"Shipyard"`
- Actual: `"Bank"`
- This appears to be a consequence of the MAP_1/MAP_2 default swap described above. With `MAP_1="Alpines"`, the `MAP_2` default has been set to `"Bank"` rather than `"Shipyard"`. This is inconsistent with PRD FR-9 which specifies `MAP_2 = "Shipyard"`. **Severity: Low — valid map name, but does not match PRD spec.**

**Passing sub-checks within TC-01-3:**
- `GAME_PRESET` rules: correct (`required|string|in:COOP,...`).
- `PORT` default: `"7777"` correct; rules include `required|numeric`.
- `NAME` env_variable: correctly `NAME` (not `SERVER_NAME`).
- `MAP_1`, `MAP_2`, `GAMETYPE_1`, `GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`: all have `nullable` in rules.
- `GAMETYPE_0`, `GAMETYPE_1`, `GAMETYPE_2` defaults: all `"R6Game.R6TerroristHuntCoopGame"`.
- `INTERNET_SERVER` and `INSTALL_OPENRVS` rules: `required|string|in:true,false`.
- All 14 variables: non-empty `name`, `description`, `env_variable`, `rules`, `field_type`.
- All `field_type` values: `"text"`.
- All `user_viewable` values: `true`.
- FR-13 (`user_editable: true` with `user_viewable: false`): no violation.

---

### TC-01-4 (Static portion): Port Declaration Check

**Description:** Confirm `PORT` variable declares default `7777`, rules contain `required|numeric`, and the description references derived ports.

**Result: PASS**

- `PORT` default is `"7777"`.
- Rules include `required|numeric` (full: `required|numeric|min:1024|max:65535`).
- Description text: *"Primary UDP game port. The server beacon port will be PORT+1000 (default 8777) and the beacon port will be PORT+2000 (default 9777). All three ports must be allocated in the panel."* — fully satisfies the requirement.

---

### TC-01-5: Fixed-Value Field Checks

**Description:** Verify exact values for all fixed-value fields listed in strategy §5.2.

**Result: FAIL**

| Field Path | Expected | Actual | Match |
|---|---|---|---|
| `_comment` | `"DO NOT EDIT: FILE GENERATED AUTOMATICALLY BY PTERODACTYL PANEL - PTERODACTYL.IO"` | Same | PASS |
| `meta.version` | `"PTDL_v2"` | `"PTDL_v2"` | PASS |
| `meta.update_url` | `null` | `null` | PASS |
| `name` | `"Raven Shield"` | `"Rainbow Six 3: Raven Shield"` | **FAIL** |
| `author` | `"michaelconne@gmail.com"` | `"michaelconne@gmail.com"` | PASS |
| `features` | `null` | `null` | PASS |
| `file_denylist` | `[]` | `[]` | PASS |
| `startup` | `"/entrypoint.sh"` | `"/entrypoint.sh"` | PASS |
| `config.startup.done` | `"OpenRVS is up to date"` | `"OpenRVS is up to date"` | PASS |
| `config.stop` | `"^C"` | `"^C"` | PASS |
| `docker_images` key | `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` | Same | PASS |
| `docker_images` value | `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` | Same | PASS |
| `scripts.installation.container` | `"ghcr.io/ptero-eggs/installers:debian"` | Same | PASS |
| `scripts.installation.entrypoint` | `"bash"` | `"bash"` | PASS |

**Failure detail:**

- **FAIL-8: `name` field value is `"Rainbow Six 3: Raven Shield"` instead of `"Raven Shield"`.**
  - The strategy §5.2 specifies the `name` field (the display name shown in the panel Nest egg list) must be exactly `"Raven Shield"`. The actual value is the full franchise-prefixed title.
  - **Severity: Low-Medium** — functionally harmless but does not match the spec. The root README heading uses the full title, so there is a reasonable interpretation that the longer form is intentional. This should be confirmed with the implementer.

---

### TC-01-6: Installation Script Static Checks

**Description:** Verify `scripts.installation.script` begins with `#!/bin/bash`, contains an informational `echo`, and contains no game-file download logic.

**Result: PASS with Observation**

- Script begins with `#!/bin/bash`: PASS.
- Script contains an `echo` with an informational message: PASS.
- Script contains no game-file download logic (no `wget`, `curl`, `apt`, etc.): PASS.

**Observation (not a failure):** The script string value in the JSON contains embedded `\r\n` line endings within the string value itself (i.e., the literal characters `\r\n` appear inside the JSON string between lines). The outer JSON file uses Unix LF line endings and is valid JSON. The embedded CRLF within the script string value is also present in `config.startup` and `config.logs`. This is an encoding style issue: when the panel writes this script to disk or executes it, the CRLF characters may or may not be handled cleanly depending on the platform. **Severity: Low — the Pterodactyl panel runs on Linux and will interpret the CRLF-embedded string; practical impact is minimal, but it is a hygiene issue that could cause subtle problems in edge cases.**

---

### TC-01-7: Encoding and Line-Ending Check

**Description:** Verify file is UTF-8 without BOM, and outer file line endings are Unix LF.

**Result: PASS**

- UTF-8 encoding: valid, no parse errors.
- BOM: absent.
- CRLF sequences in the raw JSON file bytes: 0 (the outer file uses LF only).
- Bare CR (`\r` not followed by `\n`) count: 0.

*(The CRLF characters within embedded JSON string values are part of the string content, not line endings in the JSON file structure, and do not affect this check.)*

---

### TC-02-1: README Content Completeness Check

**Description:** Verify all five required sections and their sub-items are present in `rainbow_six_3_raven_shield/README.md`.

**Result: PASS**

| Requirement | Present |
|---|---|
| File exists at `rainbow_six_3_raven_shield/README.md` | Yes |
| Server Overview section (`## Server Overview`) | Yes |
| Game name "Tom Clancy's Rainbow Six 3: Raven Shield" referenced | Yes |
| Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver` linked | Yes |
| Note that image is externally maintained | Yes |
| Note that game files download from archive.org on first startup | Yes |
| Explicit statement that no CD key or Steam login is required | Yes |
| Installation / Setup section (`## Installation / Setup`) | Yes |
| Steps to import egg into panel | Yes |
| Steps to create a server | Yes |
| Note that first start may take longer | Yes |
| Environment Variables section with 14-row table | Yes (14 rows) |
| Columns: variable name, default, allowed values, description | Yes |
| All six valid GAMETYPE strings listed | Yes |
| All 18 valid map names listed | Yes |
| Port reference table (`## Server Ports`) | Yes |
| Port 7777 — primary game traffic — PORT env var | Yes |
| Port 8777 — server beacon — PORT+1000 | Yes |
| Port 9777 — beacon — PORT+2000 | Yes |
| Known Limitations section (`## Known Limitations`) | Yes |
| External Docker image limitation noted | Yes |
| Map/gametype values not validated by panel noted | Yes |
| Changing PORT requires all three ports re-allocated noted | Yes |
| Map name case-sensitivity explicitly noted | Yes |

---

### TC-02-2: README Link Validity Check

**Description:** Verify all hyperlinks in the game README have correct syntax and relative links resolve.

**Result: PASS**

All three links in `rainbow_six_3_raven_shield/README.md`:

| Link Text | URL | Status |
|---|---|---|
| `ghcr.io/danpowell88/ravenshield_dedicatedserver` | `https://ghcr.io/danpowell88/ravenshield_dedicatedserver` | Correctly formatted; no relative resolution needed |
| archive.org | `https://archive.org` | Correctly formatted |
| OpenRVS | `https://github.com/OpenRVS/openrvs-registry` | Correctly formatted |

No relative links exist in the file. No broken Markdown link syntax (unclosed brackets, missing parentheses) detected. Live reachability of external URLs is out of scope (see Section 4 below).

---

### TC-03-1: Root README Alphabetical Order Check

**Description:** Confirm the Raven Shield entry is in the correct alphabetical position in the root README.

**Result: PASS**

- Entry found: `#### [Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)` (line 256).
- Entry immediately above: `#### [Pavlov VR](./pavlov_vr)` — "Pavlov VR" sorts before "Rainbow Six 3" alphabetically: PASS.
- Entry immediately below: `#### [Renown](./renown)` — "Renown" sorts after "Rainbow Six 3" alphabetically: PASS.
- Occurrence count: exactly 1 — no duplicates.
- No existing rows removed or reordered (file otherwise clean).

**Note:** The root README uses `####` heading format, not a pipe table. Strategy §2.3.2 references "pipe delimiters" and "header separator row" which are table-format checks. These checks are not applicable to the heading-list format used by this repository. The heading-based format is consistent with the rest of the file and is therefore valid.

---

### TC-03-2: Root README Link Format Check

**Description:** Confirm the Raven Shield row link points to the correct directory and that both required files exist within it.

**Result: PASS**

- Link target: `./rainbow_six_3_raven_shield` — matches actual directory name.
- Directory `rainbow_six_3_raven_shield/` exists: Yes.
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` exists: Yes.
- `rainbow_six_3_raven_shield/README.md` exists: Yes.
- Markdown link syntax: `[Rainbow Six 3: Raven Shield](./rainbow_six_3_raven_shield)` — well-formed, no unclosed brackets or missing parentheses.

---

## Summary of All Failures

| Failure ID | TC | Severity | Description |
|---|---|---|---|
| FAIL-1 | TC-01-3 | Low | Variable ordering: MAP/GAMETYPE slots interleaved (MAP_0, GAMETYPE_0, MAP_1, GAMETYPE_1, MAP_2, GAMETYPE_2) instead of all MAPs then all GAMETYPEs |
| FAIL-2 | TC-01-3 | Medium | `NAME` default is `"My RavenShield Server"` instead of `"Pterodactyl Raven Shield"` |
| FAIL-3 | TC-01-3 | Medium | `MAX_PLAYERS` default is `"8"` instead of `"16"` |
| FAIL-4 | TC-01-3 | High | `MAP_0` rules are `nullable\|string\|max:64` instead of `required\|string\|max:64` |
| FAIL-5 | TC-01-3 | High | `GAMETYPE_0` rules are `nullable\|string\|max:64` instead of `required\|string\|max:128` |
| FAIL-6 | TC-01-3 | Low | `ADMIN_PASSWORD` and `GAME_PASSWORD` rules use `max:32` instead of `max:64` |
| FAIL-7 | TC-01-3 | Medium | `INSTALL_OPENRVS` has `user_editable: false` instead of `true` |
| FAIL-8 | TC-01-5 | Low-Medium | `name` field is `"Rainbow Six 3: Raven Shield"` instead of `"Raven Shield"` |
| Open | TC-01-3 | Info | `MAP_1` default `"Alpines"` vs PRD `"Bank"` — pre-existing spec conflict, not a new defect; `MAP_2` default is `"Bank"` instead of `"Shipyard"` per PRD |

---

## Static Test Result Summary

| TC ID | Description | Result |
|---|---|---|
| TC-01-1 | JSON Schema Validation | PASS |
| TC-01-3 | Environment Variable Verification (Static) | **FAIL** (8 issues) |
| TC-01-4 | Port Declaration Check (Static) | PASS |
| TC-01-5 | Fixed-Value Field Checks | **FAIL** (1 issue: `name` value) |
| TC-01-6 | Installation Script Static Checks | PASS (1 observation) |
| TC-01-7 | Encoding and Line-Ending Check | PASS |
| TC-02-1 | README Content Completeness Check | PASS |
| TC-02-2 | README Link Validity Check | PASS |
| TC-03-1 | Root README Alphabetical Order Check | PASS |
| TC-03-2 | Root README Link Format Check | PASS |

**Static pass rate: 8 / 10 test cases pass (80%).**
**2 test cases FAIL, with a total of 9 distinct defects/issues.**

---

## Test Cases Requiring a Live Pterodactyl Panel

The following test cases from the strategy cannot be executed statically and require a running Pterodactyl v1.x panel and/or Docker environment:

| TC ID | Description | Blocker |
|---|---|---|
| TC-01-2 | Panel Import Test — upload egg, confirm zero import errors, verify 14 variables in panel UI, confirm NAME env_variable | Live panel |
| TC-01-2 | Confirm `/entrypoint.sh` is set as startup command in panel | Live panel |
| TC-01-3 (live) | Panel rules engine triggers `in:` validation error for invalid `GAME_PRESET` | Live panel |
| AC-4 | UDP ports 7777 / 8777 / 9777 open and accepting connections | Docker + panel |
| AC-10 | End-to-end first-run boot: game file download from archive.org, `OpenRVS is up to date` log line appears in panel console | Docker + panel |
| AC-13 | Invalid `GAME_PRESET` causes entrypoint exit with error message (entrypoint is inside Docker image) | Docker |
| AC-14 | Invalid `GAMETYPE_*` causes entrypoint warning and clean exit | Docker |
| AC-12 | PR contribution checklist panel-import line item | Live panel |

---

## Go / No-Go Recommendation

**Recommendation: NO-GO — Do not submit PR until high-severity defects are resolved.**

### Blocking issues (must fix before PR)

1. **FAIL-4 (High):** `MAP_0` rules must be `required|string|max:64`. An empty `MAP_0` will cause server startup failure; the panel must prevent blank submission.
2. **FAIL-5 (High):** `GAMETYPE_0` rules must be `required|string|max:128`. Same rationale as FAIL-4. The `max:64` constraint is also incorrect per spec.

### Should-fix issues (fix before PR, or obtain explicit sign-off waiver)

3. **FAIL-2 (Medium):** `NAME` default does not match the specified value `"Pterodactyl Raven Shield"`.
4. **FAIL-3 (Medium):** `MAX_PLAYERS` default does not match the specified value `"16"`.
5. **FAIL-7 (Medium):** `INSTALL_OPENRVS` must have `user_editable: true` per spec.
6. **MAP_2 default (Info/Low):** `MAP_2` default is `"Bank"` per the egg; PRD FR-9 specifies `"Shipyard"`. Resolve the MAP_1/MAP_2 default discrepancy (strategy §6 open question) and align both MAP_1 and MAP_2 defaults with whichever authoritative spec is confirmed.

### Low-severity (fix or accept as-is with documented rationale)

7. **FAIL-8 (Low-Medium):** `name` field `"Rainbow Six 3: Raven Shield"` vs specified `"Raven Shield"`. Confirm intent with product owner.
8. **FAIL-6 (Low):** `ADMIN_PASSWORD` / `GAME_PASSWORD` `max:32` vs spec `max:64`.
9. **FAIL-1 (Low):** Variable ordering — functional impact is nil; cosmetic alignment to spec is optional.
10. **Observation (Low):** Embedded CRLF in script and config string values — hygiene issue, no functional impact expected on Linux panels.

Once the two high-severity failures (FAIL-4 and FAIL-5) are resolved and the medium-severity failures are addressed or formally waived, a re-run of static tests should be performed before PR submission and live panel testing.

---

*End of UAT Test Report*
