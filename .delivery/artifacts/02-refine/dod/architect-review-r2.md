# Architect DoD Review (Round 2) — Raven Shield Egg PRD
**Reviewer:** Solution Architect (DoD Validator)
**Date:** 2026-04-11
**PRD:** `.delivery/artifacts/02-refine/po/prd.md` v1.1
**Prior Review:** `architect-review.md` (verdict: DONE)

---

## BLOCKING Criteria

### 1. Technical constraints fully specified (Docker image, ports, paths, startup command)
**PASS** — Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver` is named in FR-5, AC-9, and the overview. Ports 7777/8777/9777 UDP and the PORT/PORT+1000/PORT+2000 formula are specified in FR-7 and Section 3.5. Volume path `/rvs`, subdirectories `/rvs/gamefiles/` and `/rvs/setup/`, and startup command `/entrypoint.sh` are all explicitly stated. Installation container `ghcr.io/ptero-eggs/installers:debian` is named in Section 3.6. All technical constraint categories are present and internally consistent with no ambiguity.

### 2. All external dependencies named and accurate
**PASS** — All external dependencies are identified: Docker image at `ghcr.io/danpowell88/ravenshield_dedicatedserver`, archive.org as the game-file source, the OpenRVS community patch, and the installer image `ghcr.io/ptero-eggs/installers:debian`. The PRD correctly characterises the Docker image as externally maintained (FR-16, FR-20). No undocumented or speculative dependencies are introduced.

### 3. Startup command approach (/entrypoint.sh baked into image) correctly justified
**PASS** — FR-6 explicitly states that `/entrypoint.sh` is baked into the Docker image (not user-managed), identifies it as a "yolk" script, and explains why this satisfies the repository's PR checklist requirement. The Dockerfile for the image confirms `COPY entrypoint.sh ... /` and `ENTRYPOINT ["/entrypoint.sh"]`. The justification is accurate and complete.

### 4. Port allocations (7777/8777/9777 UDP, formula PORT/PORT+1000/PORT+2000) correctly specified
**PASS** — FR-7 names both derived variable names (`SERVER_BEACON_PORT`, `BEACON_PORT`) and their formulas (PORT+1000, PORT+2000) with worked examples (8777, 9777). Section 3.5 repeats the port table with protocol (UDP), purpose, and derivation columns. FR-19 requires the same table in the README. AC-4 and the success metrics table both reference all three ports. Formula matches the confirmed entrypoint.sh source.

### 5. Env var names match actual entrypoint.sh implementation (NAME not SERVER_NAME, etc.)
**PASS** — FR-9 uses `NAME` (not `SERVER_NAME`), confirmed correct against the entrypoint source. All ten env var names in FR-9 and FR-10 (`NAME`, `PORT`, `MAX_PLAYERS`, `GAME_PRESET`, `MAP_0`–`MAP_2`, `GAMETYPE_0`–`GAMETYPE_2`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`) match the variable names read by the entrypoint script. AC-5 and the success metrics table explicitly call out the `NAME` requirement as a named acceptance check.

---

## WARNING Criteria

### 6. Valid GAMETYPE values are listed
**PASS** — FR-11 enumerates six `GAMETYPE_*` string values with their human-readable mode names. FR-9 row 1 enumerates seven `GAME_PRESET` enum values in the Pterodactyl validation rule (`required|string|in:COOP,ADVERSARIAL,DEATHMATCH,TEAMDEATHMATCH,BOMB,HOSTAGERESCUE,ESCORTPILOT`). Both lists match confirmed entrypoint source values. FR-20 (Known Limitations) requires documentation that incorrect gametype values cause startup failure. One minor documentation gap persists from r1: the PRD does not map each `GAME_PRESET` value to its corresponding `GAMETYPE_*` string; this remains low-severity and does not block implementation.

### 7. Pterodactyl PTDL_v2 format requirements addressed
**PASS** — FR-4 enumerates all required top-level fields (`_comment`, `meta.version`, `meta.update_url`, `exported_at`, `name`, `author`, `description`, `features`, `docker_images`, `file_denylist`, `startup`, `config`, `scripts.installation`, `variables`) with required values or constraints for each. FR-14 mandates `field_type: "text"` for all variables. FR-13 enforces the `user_editable` + `user_viewable` consistency rule. NFR-1 requires strict JSON validity, NFR-2 requires `meta.version: "PTDL_v2"`, and AC-2 requires successful panel import. All PTDL_v2 structural requirements are fully addressed.

---

## Non-Blocking Observations (carried from r1, no new issues found)

1. **GAMETYPE / GAME_PRESET mapping gap** (low severity): The PRD does not map `GAME_PRESET` preset values to their corresponding `GAMETYPE_*` strings. Operator confusion is possible; recommend the README author include a mapping table.

2. **Stop signal (OQ-2)**: `^C` (SIGINT) is listed as the stop command with an open question about clean SIGINT handling in the entrypoint. Appropriately flagged as low priority; `^C` is a safe default for this repo.

3. **MAP slots capped at 3**: The entrypoint supports MAP_0–MAP_31 but the PRD exposes only MAP_0–MAP_2. This is a deliberate scope decision and acceptable for a baseline egg.

4. **Docker image availability (OQ-1)**: Image is externally maintained with no digest pin. FR-20 requires documenting this as a known limitation. No further action required at PRD level.

---

## Final Verdict

**DONE** — All five BLOCKING criteria PASS. Both WARNING criteria PASS. The PRD is complete, internally consistent, and correctly aligned with the entrypoint.sh implementation. No new blocking defects found in round 2. Ready for implementation handoff without changes.
