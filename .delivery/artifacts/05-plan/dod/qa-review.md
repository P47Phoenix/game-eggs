# QA DoD Review — Stage 5 Plan Artifacts

**Reviewer:** QA Engineer (delivery-team:quality)
**Date:** 2026-04-11
**Pipeline:** run-2026-04-11-r6fix
**Mode:** Light (blocking criteria only)
**Artifacts reviewed:**
- `C:/GitHub/game-eggs/.delivery/artifacts/05-plan/po/stories.md`
- `C:/GitHub/game-eggs/.delivery/artifacts/05-plan/qa/test-strategy.md`

---

## Gate Verdicts

### GATE 1 — Test strategy exists and covers Track A (JSON edits) and Track B (crash fix, both outcomes)

**PASS**

The test strategy explicitly defines:

- Section 2.1 scopes Track A (static JSON verification covering all structural, description, and rules checks) and Track B (gated sub-tracks B1 and B2, covering both the override-variable-confirmed path and the no-override-exists path).
- Track A is covered by TC-A-01 through TC-A-12 (11 static + 1 empirical).
- Track B1 is covered by TC-B1-01 through TC-B1-04 (3 static + 1 empirical, active only when Gate 1 = override confirmed).
- Track B2 is covered by TC-B2-01 through TC-B2-03 (3 static, active only when Gate 1 = no override).
- The strategy correctly gates Track B on Gate 1 outcome and requires exactly one sub-track to be active.
- The validation matrix (Section 4) maps every AC to at least one test case.

---

### GATE 2 — All acceptance criteria from the story have at least one corresponding test case

**PASS**

Cross-referencing the validation matrix against all story ACs:

| AC Range | Count | Test Case(s) | Status |
|---|---|---|---|
| AC-A-01 to AC-A-05 | 5 | TC-A-01 | Covered |
| AC-A-06 | 1 | TC-A-01 (regression) | Covered |
| AC-A-07 to AC-A-11 | 5 | TC-A-02 | Covered |
| AC-A-12 | 1 | TC-A-02 (regression) | Covered |
| AC-A-13 to AC-A-18 | 6 | TC-A-03 | Covered |
| AC-A-19 to AC-A-22 | 4 | TC-A-04 | Covered |
| AC-A-23 to AC-A-24 | 2 | TC-A-05 | Covered |
| AC-A-25 to AC-A-28 | 4 | TC-A-06 | Covered |
| AC-A-29 | 1 | TC-A-07 (static) + TC-A-12 (empirical) | Covered |
| AC-A-30 | 1 | TC-A-08 | Covered |
| AC-A-31 | 1 | TC-A-09 | Covered |
| AC-A-32 | 1 | TC-A-10 | Covered |
| AC-A-33 | 1 | TC-A-11 | Covered |
| AC-B1-01 | 1 | TC-B1-01 | Covered (B1 gated) |
| AC-B1-02 | 1 | TC-B1-01 | Covered (B1 gated) |
| AC-B1-03 | 1 | TC-B1-02 | Covered (B1 gated) |
| AC-B1-04 | 1 | TC-B1-04 | Covered (B1 gated, empirical) |
| AC-B1-05 | 1 | TC-B1-03 | Covered (B1 gated) |
| AC-B2-01 | 1 | TC-B2-01 | Covered (B2 gated) |
| AC-B2-02 | 1 | TC-B2-01 | Covered (B2 gated) |
| AC-B2-03 | 1 | TC-B2-02 | Covered (B2 gated) |
| AC-B2-04 | 1 | TC-B2-03 | Covered (B2 gated) |

All 46 ACs have at least one mapped test case. No AC is orphaned.

---

### GATE 3 — Invalid-GAMETYPE error case has an explicit test case

**PASS**

AC-A-29 (invalid GAMETYPE causes non-zero exit and error message before Wine launch) is addressed by two explicit test cases:

- **TC-A-07** (static): Verifies that all six GAMETYPE slot descriptions contain warning text indicating invalid values cause a startup exit with error. Steps assert presence of semantically equivalent keywords ("invalid", "exit", "error", "startup") in every GAMETYPE_0 through GAMETYPE_5 description. Coverage is not limited to GAMETYPE_0 — all six slots are checked, satisfying the memory lesson that error-case ACs are required for enum env vars.
- **TC-A-12** (empirical): Executes `docker run --rm -e GAMETYPE_0=R6Game.InvalidType ghcr.io/danpowell88/ravenshield_dedicatedserver`, asserts non-zero exit code, and asserts `validate_gametype` error output precedes any Wine invocation lines.

The strategy correctly marks TC-A-07 as mandatory in all cases and TC-A-12 as conditionally deferrable (Docker unavailable) with required documentation of any deferral.

---

### GATE 4 — docker_images key-equals-value rule has a test case

**PASS**

AC-A-32 is addressed by **TC-A-10**:

- Step 2 asserts exactly one key exists in the `docker_images` object.
- Step 3 asserts the key is `"ghcr.io/danpowell88/ravenshield_dedicatedserver"`.
- Step 4 asserts the value equals the key (i.e., the full URI is both the key and the value).

The key-equals-value constraint is encoded as a distinct explicit assertion, not merely a presence check.

---

### GATE 5 — MAP_0/GAMETYPE_0 required vs. MAP_1–5/GAMETYPE_1–5 nullable has a test case

**PASS**

AC-A-19 through AC-A-22 are addressed by **TC-A-04**:

- Step 1: Asserts `MAP_0` rules begin with `"required"`.
- Step 2: Asserts `GAMETYPE_0` rules begin with `"required"`.
- Steps 3–4: Assert `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` rules each begin with `"nullable"`.
- Steps 5–6: Negative assertions confirm no slot-0 begins with `nullable` and no slot-1+ begins with `required`.

TC-A-06 (new slot field values) provides a cross-check by asserting `nullable` for all six new variables. The memory lesson (MAP/GAMETYPE slot 0 required; slots 1+ nullable) is fully satisfied.

---

## Summary

| Gate | Verdict |
|---|---|
| Test strategy covers Track A and Track B (both outcomes) | PASS |
| All ACs have at least one test case | PASS |
| Invalid-GAMETYPE error case has an explicit test case | PASS |
| docker_images key-equals-value rule has a test case | PASS |
| MAP_0/GAMETYPE_0 required vs. slots 1–5 nullable has a test case | PASS |

**Overall QA DoD verdict: PASS — all five blocking criteria satisfied. Stage 5 plan artifacts are clear to proceed to Stage 6 (implementation).**

---

*End of QA DoD Review*
