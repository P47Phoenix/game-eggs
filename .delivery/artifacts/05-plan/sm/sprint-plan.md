# Sprint Plan: STORY-01 — R6 Raven Shield Egg Bug Fix & Usability

**Date:** 2026-04-11
**Sprint type:** Bug Fix (light ceremony)
**Target file:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**Optional output:** `rainbow_six_3_raven_shield/README.md` (Track B2 only)

---

## Sprint Goal

Fix unhelpful map/gametype cross-references, normalise gametype field rules, expand rotation slots to 6, and address the `/rvs` startup crash (gated on image maintainer confirmation). Deliverable is a single modified egg JSON file plus an optional README update.

---

## Task Breakdown

### Track A — Unconditional JSON edits (no gate required)

All tasks are pure egg JSON edits. Begin immediately; no blocking dependencies.

| ID | Task | AC covered |
|----|------|------------|
| A-1 | Expand `MAP_1`–`MAP_5` descriptions to contain the full inline 18-name map list. Remove all cross-reference phrases. | AC-A-01 to AC-A-05 |
| A-2 | Verify `MAP_0` description is unchanged and still contains the full map list (no regression). | AC-A-06 |
| A-3 | Expand `GAMETYPE_1`–`GAMETYPE_5` descriptions to contain all six game type values with human-readable labels inline. Remove cross-references. Add warning that invalid values cause entrypoint exit with error before any Wine process launches. | AC-A-07 to AC-A-11, AC-A-29 |
| A-4 | Verify `GAMETYPE_0` description is unchanged, lists all six values with labels, and includes the error-exit warning. | AC-A-12, AC-A-29 |
| A-5 | Normalise all `GAMETYPE_*` rules to `max:128` (fix `max:64` on slots 1 and 2). | AC-A-13 to AC-A-18 |
| A-6 | Set `MAP_0` and `GAMETYPE_0` rules to begin with `required`. Set `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` rules to begin with `nullable`. | AC-A-19 to AC-A-22 |
| A-7 | Add six new variable objects: `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5`. Each must have: `default_value: ""`, rules beginning with `nullable|string|max:128`, `user_viewable: true`, `user_editable: true`, `field_type: "text"`. Include full inline descriptions matching A-1/A-3 text. | AC-A-23 to AC-A-28 |
| A-8 | Structural integrity check: `python -m json.tool` exits 0; `meta.version` = `"PTDL_v2"`; `docker_images` key equals value equals `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`; all unmodified variables are byte-for-byte unchanged. | AC-A-30 to AC-A-33 |

**Memory constraints applied:**
- `docker_images` key must equal value (full image URI, no shorthand).
- Slot 0 (`MAP_0`, `GAMETYPE_0`) rules begin with `required`. Slots 1–5 begin with `nullable`.
- `meta.version` must remain `"PTDL_v2"`. No user-managed shell scripts.

---

### Gate 1 — Blocking dependency for Track B

**Owner:** PO / developer
**Action:** Contact image maintainer at `https://github.com/danpowell88/ravenshield_dedicatedserver` to confirm whether the image exposes an environment variable (e.g. `GAMEFILES_DIR`, `DATA_DIR`) that overrides the hardcoded `/rvs` data path.
**Required output:** Document the Gate 1 outcome (confirmed / not confirmed, plus variable name if applicable) in the delivery brief before any Track B task begins.

**Track B dev must not start until Gate 1 is resolved.**

---

### Track B — Gated on Gate 1 (implement exactly one sub-track)

#### Track B1 — If Gate 1 = "override variable confirmed"

| ID | Task | AC covered |
|----|------|------------|
| B1-1 | Add the confirmed override variable (e.g. `GAMEFILES_DIR`) as a new panel variable: `default_value` = `/home/container/gamefiles` (or maintainer-confirmed path), `user_viewable: true`, `user_editable: true`, `field_type: "text"`, `rules: "required|string|max:256"`. | AC-B1-01, AC-B1-02 |
| B1-2 | Write the variable `description` to explain: redirects game files from `/rvs` to a Pterodactyl-compatible path; warn that changing to a non-writable path breaks the server. | AC-B1-03 |
| B1-3 | Integration test (Docker available locally): `docker run --rm --user 1000:1000 -e <VAR>=<path> ghcr.io/danpowell88/ravenshield_dedicatedserver` — confirm no immediate "Permission denied" exit. | AC-B1-04, TC-10 |
| B1-4 | Confirm egg `description` field contains no operator warning about `/rvs` permission issue (unnecessary once override resolves the crash). | AC-B1-05 |

#### Track B2 — If Gate 1 = "no override exists"

| ID | Task | AC covered |
|----|------|------------|
| B2-1 | Prepend a prominent operator warning to the egg `description` field covering: (a) immediate startup crash with "Permission denied"; (b) root cause — image hardcodes `/rvs`, container runs as uid 1000; (c) image-layer defect not fixable by egg JSON alone; (d) Wings bind-mount workaround steps. Warning must appear before any other content in the field. | AC-B2-01, AC-B2-02, AC-B2-03 |
| B2-2 | Add or update `rainbow_six_3_raven_shield/README.md` with a "Known Issue: Startup Crash" section: exit code 1 symptom, root cause, bind-mount workaround step-by-step. | AC-B2-04 |
| B2-3 | Verify `python -m json.tool` exits 0 after description edit (regression guard). | AC-A-30 |

---

## Dependencies

```
[Track A: tasks A-1 through A-8]  — no blocking dependencies; start immediately
                |
            [Gate 1]  — PO contacts image maintainer; outcome documented in delivery brief
                |
         +------+------+
         |             |
    [Track B1]    [Track B2]   — implement exactly ONE based on Gate 1 outcome
```

---

## Definition of Done

### Track A (all items required)

- [ ] `MAP_1`–`MAP_5` descriptions contain the full 18-name map list inline; no cross-references remain.
- [ ] `MAP_0` description unchanged (no regression).
- [ ] `GAMETYPE_1`–`GAMETYPE_5` descriptions contain all six game type values with labels inline; no cross-references remain; all slots warn that invalid values cause entrypoint exit with error.
- [ ] `GAMETYPE_0` description unchanged (no regression).
- [ ] All six `GAMETYPE_*` rules contain `max:128`; none contain `max:64`.
- [ ] `MAP_0` and `GAMETYPE_0` rules begin with `required`; `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` rules begin with `nullable`.
- [ ] `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5` present in variables array with `default_value: ""`, `nullable`, `user_viewable: true`, `user_editable: true`, `field_type: "text"`.
- [ ] `python -m json.tool egg-rainbow-six-3-raven-shield.json` exits 0.
- [ ] `meta.version` = `"PTDL_v2"`.
- [ ] `docker_images` entry unchanged: key equals value equals `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`.
- [ ] All unmodified variables are byte-for-byte identical to pre-change values.
- [ ] TC-01 through TC-09 pass.
- [ ] Peer code review approved.

### Track B1 (if Gate 1 = override confirmed)

- [ ] Gate 1 outcome documented in delivery brief.
- [ ] Override variable added with correct default, rules, and description.
- [ ] `docker run --rm --user 1000:1000` integration test passes locally (no immediate "Permission denied" exit).
- [ ] Egg `description` contains no `/rvs` permission warning.
- [ ] TC-10 passes.
- [ ] `python -m json.tool` exits 0 after changes.

### Track B2 (if Gate 1 = no override)

- [ ] Gate 1 outcome documented in delivery brief.
- [ ] Operator warning prepended to egg `description` covering all four required points.
- [ ] Warning appears before all other content in the `description` field.
- [ ] `README.md` "Known Issue: Startup Crash" section present with symptom, root cause, and workaround steps.
- [ ] Egg `description` does not claim the crash is fixed.
- [ ] TC-11 passes.
- [ ] `python -m json.tool` exits 0 after description edit.

---

## Deliverables

1. `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` — modified (all tracks)
2. `rainbow_six_3_raven_shield/README.md` — new or updated (Track B2 only)
