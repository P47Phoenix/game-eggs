# PO DoD Review — Planning Stage
**Date:** 2026-04-11
**Reviewer:** Product Owner (DoD Validator)
**Stories:** `.delivery/artifacts/05-plan/po/stories.md`
**PRD:** `.delivery/artifacts/02-refine/po/prd.md`

---

## Blocking Criteria Evaluation

### 1. Every PRD functional requirement is covered by at least one story

| PRD Requirement | Covered By | Status |
|---|---|---|
| FR-1: Directory `rainbow_six/raven_shield/` | STORY-01 (AC-01-1 uses `rainbow_six_3_raven_shield/` — path diverges from PRD) | PARTIAL — see finding F-1 |
| FR-2: Egg JSON + README.md in directory | STORY-01 + STORY-02 | PASS |
| FR-3: Root README row linking to `rainbow_six/raven_shield/` | STORY-03 (AC-03-4 references `rainbow_six_3_raven_shield/` — diverges) | PARTIAL — see finding F-1 |
| FR-4: PTDL_v2 schema, required top-level fields | STORY-01 AC-01-3, AC-01-4, AC-01-5, AC-01-20 | PASS |
| FR-5: Exact docker_images entry | STORY-01 AC-01-6 | PASS |
| FR-6: `startup` = `/entrypoint.sh` | STORY-01 AC-01-7 | PASS |
| FR-7: PORT derivation (7777, +1000, +2000) | STORY-01 AC-01-15; STORY-02 AC-02-7 | PASS |
| FR-8: `config.startup.done` = `"OpenRVS is up to date"` | STORY-01 AC-01-8 | PASS |
| FR-9: Core env vars (10 variables) | STORY-01 AC-01-10 through AC-01-15 | PASS |
| FR-10: Optional env vars (4 variables) | STORY-01 AC-01-10 | PASS |
| FR-11: Valid GAMETYPE_* values | STORY-02 AC-02-5 | PASS |
| FR-12: Valid map names | STORY-02 AC-02-6 | PASS |
| FR-13: No user_editable:true with user_viewable:false | STORY-01 AC-01-18 | PASS |
| FR-14: All field_type = "text" | STORY-01 AC-01-17 | PASS |
| FR-15: config.stop = "^C" | STORY-01 AC-01-9 | PASS |
| FR-16: README server overview (Docker image, archive.org, no CD key) | STORY-02 AC-02-2 | PASS |
| FR-17: README installation instructions | STORY-02 AC-02-3 | PASS |
| FR-18: README env var reference table (14 vars + gametype + map lists) | STORY-02 AC-02-4, AC-02-5, AC-02-6 | PASS |
| FR-19: README port reference table | STORY-02 AC-02-7 | PASS |
| FR-20: README known limitations (3 items) | STORY-02 AC-02-8 | PASS |
| FR-21: Root README row, alphabetical, correct link | STORY-03 AC-03-1 through AC-03-6 | PASS (subject to F-1) |
| NFR-1: JSON validity | STORY-01 AC-01-2 | PASS |
| NFR-2: PTDL_v2 panel compatibility | STORY-01 AC-01-4, TC-01-2 | PASS |
| NFR-3: PR checklist compliance | STORY-01 DoD, STORY-02 DoD, STORY-03 DoD | PASS |
| NFR-4: No bundled binaries | Covered implicitly by no-op install script (AC-01-16) | PASS |
| NFR-5: UTF-8 Unix line endings | STORY-01 AC-01-19 | PASS |
| Section 3.6 (Installation script: no-op, container, entrypoint) | STORY-01 AC-01-16 | PASS |

### 2. All 3 deliverables represented as stories

| Deliverable | Story | Status |
|---|---|---|
| Egg JSON (`egg-*.json`) | STORY-01 | PASS |
| Game directory README.md | STORY-02 | PASS |
| Root README.md update | STORY-03 | PASS |

### 3. Story acceptance criteria trace back to PRD acceptance criteria

Traceability reviewed for all stories:

- STORY-01 ACs explicitly reference FR-4, FR-5, FR-6, FR-8, FR-9, FR-10, FR-13, FR-14, FR-15, FR-16, NFR-1, NFR-5, and PRD ACs AC-5, AC-6, AC-9, AC-11. Full traceability confirmed.
- STORY-02 ACs explicitly reference FR-2, FR-11, FR-12, FR-16, FR-17, FR-18, FR-19, FR-20, and PRD ACs AC-7a through AC-7e. Full traceability confirmed.
- STORY-03 ACs explicitly reference FR-21 and PRD AC-8. Full traceability confirmed.

PRD acceptance criteria coverage:

| PRD AC | Covered By Story AC | Status |
|---|---|---|
| AC-1 (JSON validity, correct path) | STORY-01 AC-01-2; path divergence noted in F-1 | PARTIAL |
| AC-2 (panel import, 14 vars) | STORY-01 TC-01-2 | PASS |
| AC-3 (startup = /entrypoint.sh) | STORY-01 AC-01-7 | PASS |
| AC-4 (ports 7777/8777/9777 open) | STORY-02 AC-02-7; no explicit story-level AC about ports being open/allocated in panel | GAP — see F-2 |
| AC-5 (14 vars, NAME not SERVER_NAME) | STORY-01 AC-01-10, AC-01-11 | PASS |
| AC-6 (author email) | STORY-01 AC-01-5 | PASS |
| AC-7 (README sections a–e) | STORY-02 AC-02-2 through AC-02-8 | PASS |
| AC-8 (root README row, alphabetical) | STORY-03 AC-03-1 through AC-03-5 | PASS |
| AC-9 (docker_images exact entry) | STORY-01 AC-01-6 | PASS |
| AC-10 (end-to-end first boot, archive.org, OpenRVS string) | No story-level AC covers the end-to-end live boot path | GAP — see F-3 |
| AC-11 (config.startup.done value) | STORY-01 AC-01-8 | PASS |
| AC-12 (PR checklist 100%) | STORY-01/02/03 DoD items | PASS |
| AC-13 (invalid GAME_PRESET rejected by entrypoint with exit 1) | STORY-01 TC-01-3 covers panel validation; entrypoint-level exit-1 behaviour not in AC | GAP — see F-4 |
| AC-14 (invalid GAMETYPE_* entrypoint warning + exit) | No story-level AC or TC covers this | GAP — see F-4 |

### 4. No PRD requirement left unimplemented

All functional requirements are addressed by at least one story. Four gaps are noted in the acceptance criteria traceability (not missing stories, but missing ACs within existing stories).

---

## Findings

**F-1 — Directory path inconsistency (BLOCKING)**
PRD FR-1, FR-2, FR-3, AC-1, and AC-8 specify the directory as `rainbow_six/raven_shield/` and the egg file as `egg-raven-shield.json`. The stories use `rainbow_six_3_raven_shield/` and `egg-rainbow-six-3-raven-shield.json` throughout. This is a factual inconsistency between the PRD and the stories. Either the PRD must be updated to reflect the chosen path, or the stories must be corrected. Implementation risk: if the stories are implemented as written, the root README link in STORY-03 will not match the PRD's specified path.

**F-2 — PRD AC-4 (ports open/allocated in panel) not covered by a story AC (MINOR)**
PRD AC-4 requires that UDP ports 7777, 8777, and 9777 are all open and accepting traffic on a running server. The stories document the ports in the README (STORY-02) but no story acceptance criterion requires or tests that all three ports are declared in the egg's panel port configuration or verified as open on a live instance. This is a documentation-only gap; the PRD section 3.5 requirement is implicitly handled by the Docker image's entrypoint, but no story AC captures it.

**F-3 — PRD AC-10 (end-to-end first-boot test) not covered by a story AC (MINOR)**
PRD AC-10 is an end-to-end live-boot acceptance criterion (archive.org download, OpenRVS patch, console string). No story-level AC or test case explicitly covers this end-to-end scenario. STORY-01 TC-01-2 covers panel import only. A test case for first-boot behaviour would strengthen coverage.

**F-4 — PRD AC-13 and AC-14 (entrypoint-level validation with exit codes) not fully covered (MINOR)**
PRD AC-13 requires the entrypoint to exit with code 1 on an invalid GAME_PRESET. STORY-01 TC-01-3 only covers panel-level validation (the `in:` rule), not the entrypoint's runtime behaviour. PRD AC-14 (invalid GAMETYPE_* causes entrypoint warning and exit) has no corresponding story AC or test case at all. Since the entrypoint is part of an external Docker image (out of scope per Section 6), this may be intentionally untestable — but the stories do not document this scoping decision.

---

## Verdict

**NOT_DONE**

The planning artifacts cannot be signed off in their current state due to **F-1**, which is a blocking path inconsistency between the PRD and all three stories. The directory name, egg file name, and root README link path differ between documents. This must be resolved — either by updating the PRD to adopt the story paths, or by correcting the stories to match the PRD paths — before implementation begins.

Findings F-2, F-3, and F-4 are non-blocking but should be addressed (or explicitly acknowledged as out-of-scope) in the stories before the sprint starts.

**Required action before DONE:**
1. Resolve F-1: align directory and file names across PRD and stories (single source of truth).
2. Disposition F-4: either add ACs for entrypoint-level validation behaviour or explicitly note in the stories that AC-13/AC-14 are not implementable because the entrypoint is part of an external image (out of scope per PRD Section 6).
3. Optionally address F-2 and F-3 by adding test cases or noting the limitation.
