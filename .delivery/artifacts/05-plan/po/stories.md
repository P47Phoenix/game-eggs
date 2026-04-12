# User Stories: Rainbow Six 3: Raven Shield — Startup Crash Bug Fix & Usability Enhancements

**Date:** 2026-04-11
**Author:** Michael Connelly (michaelconne@gmail.com)
**Pipeline run:** run-2026-04-11-r6fix
**Type:** BUG_FIX (with bundled usability enhancements)
**Target file:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## STORY-01: Egg Usability Enhancements and Crash-Fix (Two-Track)

### Title
Fix unhelpful map/gametype cross-references, normalise gametype field rules, expand rotation slots to 6, and address the /rvs startup crash (gated on image maintainer confirmation)

### User Story

**As a** Pterodactyl panel administrator setting up a Rainbow Six 3: Raven Shield server,
**I want** the egg to display the full list of valid map names and game type values inline on every rotation slot, to expose six rotation slots instead of three, and — if the upstream Docker image supports it — to redirect game files to `/home/container` so the server starts without immediately crashing,
**So that** I can configure a working map rotation without cross-referencing another field, and so that operators are not silently blocked by a permission failure on `/rvs` with no diagnostic output.

---

### Track A — Unconditional (no gate required)

These changes are deliverable by egg JSON edits alone, regardless of the Gate 1 outcome.

#### Acceptance Criteria — Map Descriptions (Track A)

**AC-A-01.** `MAP_1` description contains the full inline list of valid map names:
Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse.
The description must not read "See MAP_0 for valid map names" or any equivalent cross-reference.

**AC-A-02.** `MAP_2` description contains the same full inline list of valid map names as AC-A-01.
The description must not read "See MAP_0 for valid map names" or any equivalent cross-reference.

**AC-A-03.** `MAP_3` description contains the same full inline list of valid map names as AC-A-01.

**AC-A-04.** `MAP_4` description contains the same full inline list of valid map names as AC-A-01.

**AC-A-05.** `MAP_5` description contains the same full inline list of valid map names as AC-A-01.

**AC-A-06.** `MAP_0` description is preserved as-is (it already contains the full inline list; no regression).

#### Acceptance Criteria — Game Type Descriptions (Track A)

**AC-A-07.** `GAMETYPE_1` description contains all six valid game type values with human-readable labels, listed inline:
- `R6Game.R6TerroristHuntCoopGame` — Terrorist Hunt Co-op
- `R6Game.R6TeamBomb` — Team Bomb
- `R6Game.R6HostageRescueAdvGame` — Hostage Rescue
- `R6Game.R6TeamDeathMatchGame` — Team Deathmatch
- `R6Game.R6EscortPilotGame` — Escort the Pilot
- `R6Game.R6DeathMatch` — Deathmatch

The description must not read "See GAMETYPE_0 for valid values" or any equivalent cross-reference.

**AC-A-08.** `GAMETYPE_2` description contains the same full inline game type list and labels as AC-A-07.
The description must not read "See GAMETYPE_0 for valid values" or any equivalent cross-reference.

**AC-A-09.** `GAMETYPE_3` description contains the same full inline game type list and labels as AC-A-07.

**AC-A-10.** `GAMETYPE_4` description contains the same full inline game type list and labels as AC-A-07.

**AC-A-11.** `GAMETYPE_5` description contains the same full inline game type list and labels as AC-A-07.

**AC-A-12.** `GAMETYPE_0` description is preserved and continues to list all six valid values with human-readable labels (no regression).

#### Acceptance Criteria — GAMETYPE Rules Normalisation (Track A)

**AC-A-13.** `GAMETYPE_0` rules string contains `max:128`.

**AC-A-14.** `GAMETYPE_1` rules string contains `max:128` (previously `max:64` — this must change).

**AC-A-15.** `GAMETYPE_2` rules string contains `max:128` (previously `max:64` — this must change).

**AC-A-16.** `GAMETYPE_3` rules string contains `max:128`.

**AC-A-17.** `GAMETYPE_4` rules string contains `max:128`.

**AC-A-18.** `GAMETYPE_5` rules string contains `max:128`.

#### Acceptance Criteria — Slot 0 Required / Slots 1–5 Nullable (Track A)

**AC-A-19.** `MAP_0` rules string begins with `required` (not `nullable`).

**AC-A-20.** `GAMETYPE_0` rules string begins with `required` (not `nullable`).

**AC-A-21.** `MAP_1` through `MAP_5` rules strings each begin with `nullable` (not `required`).

**AC-A-22.** `GAMETYPE_1` through `GAMETYPE_5` rules strings each begin with `nullable` (not `required`).

#### Acceptance Criteria — Rotation Slot Expansion to 6 (Track A)

**AC-A-23.** The `variables` array contains `MAP_3`, `MAP_4`, and `MAP_5` env variables (three new map slots added beyond the existing MAP_0–MAP_2).

**AC-A-24.** The `variables` array contains `GAMETYPE_3`, `GAMETYPE_4`, and `GAMETYPE_5` env variables (three new gametype slots added beyond the existing GAMETYPE_0–GAMETYPE_2).

**AC-A-25.** Each new map slot (`MAP_3`, `MAP_4`, `MAP_5`) has `default_value` of `""` (empty string) and `nullable` in its rules.

**AC-A-26.** Each new gametype slot (`GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`) has `default_value` of `""` (empty string) and `nullable` in its rules.

**AC-A-27.** All new map and gametype slots have `user_viewable: true` and `user_editable: true`.

**AC-A-28.** All new variables have `field_type: "text"` (PTDL_v2 has no native dropdown/select type).

#### Acceptance Criteria — Invalid GAMETYPE Error Case (Track A — testable from entrypoint)

**AC-A-29.** Given a container started with an invalid `GAMETYPE_0` value (e.g., `GAMETYPE_0=R6Game.InvalidType`), when the entrypoint's `validate_gametype` function runs, then the container exits with a non-zero exit code and produces an error message before any Wine process is launched. This behaviour is confirmed from the `validate_gametype` function in `entrypoint.sh`; the egg description for all GAMETYPE slots must warn that invalid values cause the server to exit with an error at startup.

#### Acceptance Criteria — Structural Integrity (Track A)

**AC-A-30.** `python -m json.tool egg-rainbow-six-3-raven-shield.json` exits with code 0 after all Track A changes are applied (valid JSON).

**AC-A-31.** `meta.version` remains `"PTDL_v2"`.

**AC-A-32.** `docker_images` key equals value equals `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` (no change from current — no regression).

**AC-A-33.** All existing variables not modified by Track A (GAME_PRESET, PORT, NAME, MAX_PLAYERS, MAP_0, GAMETYPE_0, ADMIN_PASSWORD, GAME_PASSWORD, INTERNET_SERVER, INSTALL_OPENRVS) retain their current `name`, `description`, `default_value`, `rules`, and `field_type` values unchanged.

---

### Track B — Gated on Gate 1 (image-path confirmation)

Gate 1 question: Does `ghcr.io/danpowell88/ravenshield_dedicatedserver` expose an environment variable (e.g., `GAMEFILES_DIR`, `DATA_DIR`, or similar) that overrides the hardcoded `/rvs` data path?

Exactly one of the two sub-tracks below applies. The developer must not implement both. The Gate 1 outcome must be documented in the delivery brief before Stage 2 begins.

#### Track B1 — If Gate 1 resolves "override variable confirmed"

**AC-B1-01.** The egg exposes the confirmed environment variable (e.g., `GAMEFILES_DIR`) as a panel variable with `default_value` pointing to `/home/container/gamefiles` (or the path confirmed by the maintainer as compatible with `/home/container`).

**AC-B1-02.** The new variable has `user_viewable: true`, `user_editable: true`, `field_type: "text"`, and `rules: "required|string|max:256"` (or a stricter rule if appropriate for the path format).

**AC-B1-03.** The new variable's description explains its purpose: it redirects game files from the default `/rvs` path to a Pterodactyl-compatible path under `/home/container`, and warns that changing this value may break the server if the path is not writable by the container user.

**AC-B1-04.** Given a container started via `docker run --rm --user 1000:1000` with the new variable set to a writable path, when the entrypoint runs, then the container does not immediately exit with "Permission denied" on the `mkdir` call (startup crash is resolved at the egg level).

**AC-B1-05.** The egg `description` field does not contain any operator warning about the `/rvs` permission issue (the warning is unnecessary if the override variable resolves the crash).

#### Track B2 — If Gate 1 resolves "no override exists" (path is hardcoded)

**AC-B2-01.** The egg `description` field contains a prominent operator warning explaining:
  (a) the server container will crash immediately on startup with "Permission denied" when run under Pterodactyl;
  (b) the root cause is that the Docker image writes to `/rvs` which is owned by root, and the container runs as uid 1000;
  (c) this is an image-layer defect — it cannot be resolved by egg JSON changes alone;
  (d) the Wings bind-mount workaround: mount the Pterodactyl server data volume at `/rvs` in the Wings node's Docker configuration to resolve the permission error.

**AC-B2-02.** The warning in the `description` field is placed before any other content in that field so it is visible at the top of the panel's egg description display.

**AC-B2-03.** The egg `description` field does not claim the startup crash has been fixed.

**AC-B2-04.** If the repository has a `README.md` for the egg, it contains a dedicated "Known Issue: Startup Crash" section that:
  (a) states the container crashes on startup with exit code 1 and "Permission denied" output;
  (b) identifies the root cause (Docker image hardcodes `/rvs`; Pterodactyl mounts at `/home/container`; uid 1000 cannot write to `/rvs`);
  (c) provides the Wings bind-mount workaround step-by-step.

---

### Test Cases

**TC-01 (Track A — happy path, map description inline content):**
Given the updated egg JSON, when `MAP_1`, `MAP_2`, `MAP_3`, `MAP_4`, and `MAP_5` descriptions are inspected, then each contains the literal string "Warehouse" (the last item in the valid map list) and does not contain the phrase "See MAP_0". `MAP_0` description is unchanged and also contains "Warehouse".

**TC-02 (Track A — happy path, gametype description inline content):**
Given the updated egg JSON, when `GAMETYPE_1` through `GAMETYPE_5` descriptions are inspected, then each contains "R6Game.R6DeathMatch" and "Deathmatch" and does not contain the phrase "See GAMETYPE_0". `GAMETYPE_0` description is unchanged and also contains all six values.

**TC-03 (Track A — gametype max:128 normalisation):**
Given the updated egg JSON, when the `rules` field for each of `GAMETYPE_0` through `GAMETYPE_5` is inspected, then every value contains the substring `max:128` and none contain `max:64`.

**TC-04 (Track A — slot required/nullable enforcement):**
Given the updated egg JSON, when `MAP_0` and `GAMETYPE_0` rules are inspected, then both begin with `required`. When `MAP_1` through `MAP_5` and `GAMETYPE_1` through `GAMETYPE_5` rules are inspected, then all ten begin with `nullable`.

**TC-05 (Track A — 6 rotation slots present):**
Given the updated egg JSON, when the `variables` array is inspected, then variables with `env_variable` values `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5` are all present. Total variable count is the prior count plus 6 (new slots only; no existing variables removed).

**TC-06 (Track A — new slots have empty defaults and correct flags):**
Given `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5` in the variables array, when each is inspected, then `default_value` is `""`, `user_viewable` is `true`, `user_editable` is `true`, and `field_type` is `"text"`.

**TC-07 (Track A — invalid GAMETYPE causes exit with error):**
Given a container launched with `docker run --rm -e GAMETYPE_0=R6Game.NotARealType ghcr.io/danpowell88/ravenshield_dedicatedserver`, when the container exits, then the exit code is non-zero and the container output contains an error message from `validate_gametype` before any Wine invocation output appears. (Confirms the AC-A-29 warning in the description is warranted and accurate.)

**TC-08 (Track A — JSON validity after all changes):**
Given the modified egg JSON file, when `python -m json.tool egg-rainbow-six-3-raven-shield.json` is executed, then the command exits with code 0 and produces no errors.

**TC-09 (Track A — no regression on unmodified variables):**
Given the updated egg JSON, when GAME_PRESET, PORT, NAME, MAX_PLAYERS, ADMIN_PASSWORD, GAME_PASSWORD, INTERNET_SERVER, and INSTALL_OPENRVS are inspected, then each variable's `name`, `description`, `default_value`, `rules`, and `field_type` are byte-for-byte identical to the values in the pre-change egg file.

**TC-10 (Track B1 — override variable resolves startup crash, if applicable):**
Given the egg exposes `GAMEFILES_DIR` (or the confirmed variable name) with default `/home/container/gamefiles`, when `docker run --rm --user 1000:1000 -e GAMEFILES_DIR=/home/container/gamefiles ghcr.io/danpowell88/ravenshield_dedicatedserver` is executed, then the container does not exit immediately with "Permission denied" and produces at least one line of entrypoint output before any failure.

**TC-11 (Track B2 — warning present in description, if applicable):**
Given no override variable exists and Track B2 is active, when the egg `description` field is read, then it contains the words "Permission denied", "uid 1000" (or equivalent), and "bind-mount" (or "Wings") as the workaround indicator, and this content appears before the existing server description prose.

---

### Definition of Done

**Track A (all items required):**
- [ ] AC-A-01 through AC-A-05: MAP_1–MAP_5 descriptions contain full inline map list
- [ ] AC-A-06: MAP_0 description unchanged (no regression)
- [ ] AC-A-07 through AC-A-11: GAMETYPE_1–GAMETYPE_5 descriptions contain full inline gametype list with labels
- [ ] AC-A-12: GAMETYPE_0 description unchanged (no regression)
- [ ] AC-A-13 through AC-A-18: All GAMETYPE_* rules use max:128
- [ ] AC-A-19 through AC-A-22: Slot 0 required; slots 1–5 nullable
- [ ] AC-A-23 through AC-A-28: MAP_3–MAP_5 and GAMETYPE_3–GAMETYPE_5 added with empty defaults, nullable, user_viewable/editable true, field_type text
- [ ] AC-A-29: GAMETYPE description for all slots warns that invalid values cause entrypoint exit with error
- [ ] AC-A-30: JSON valid (python -m json.tool exits 0)
- [ ] AC-A-31: meta.version remains PTDL_v2
- [ ] AC-A-32: docker_images entry unchanged
- [ ] AC-A-33: All unmodified variables are byte-for-byte identical to pre-change values
- [ ] TC-01 through TC-09 pass
- [ ] Peer code review approved

**Track B (exactly one sub-track required — check one):**
- [ ] **Track B1:** AC-B1-01 through AC-B1-05 verified (Gate 1 = override variable confirmed); TC-10 passes
- [ ] **Track B2:** AC-B2-01 through AC-B2-04 verified (Gate 1 = no override exists); TC-11 passes

---

### Estimated Complexity

**S** (Small) — All Track A changes are mechanical egg JSON edits (description text, rules strings, new variable objects). Track B complexity depends on Gate 1 outcome: B1 adds one variable (XS); B2 adds description and README text (XS). Primary risk is ensuring all six new GAMETYPE descriptions are complete and no cross-references are left behind.

### Dependencies

- **Gate 1** must be resolved by the PO (contact image maintainer at `https://github.com/danpowell88/ravenshield_dedicatedserver`) before Track B implementation begins. Track A has no blocking dependencies and can be implemented immediately.
- **Gate 2** (QA pre-merge): QA must verify the archive.org game files URL is live and the file is accessible before the PR is merged. A dead URL is a separate upstream issue and does not block Track A changes from merging.
