# QA DoD Review — Raven Shield Egg PRD v2.0

**Reviewed:** 2026-04-12
**Reviewer role:** QA Engineer / DoD Validator
**Artifact reviewed:** `.delivery/artifacts/02-refine/po/prd.md` (v2.0)

---

## Criterion 1: Every requirement has testable acceptance criteria

**PASS**

All 6 requirements (REQ-1 through REQ-6) have explicit, numbered acceptance criteria:
- REQ-1: AC-1.1 through AC-1.4
- REQ-2: AC-2.1 through AC-2.7
- REQ-3: AC-3.1 through AC-3.8
- REQ-4: AC-4.1 through AC-4.8
- REQ-5: AC-5.1 through AC-5.6
- REQ-6: AC-6.1 through AC-6.4

Each AC is binary (pass/fail) and references a specific observable behavior or verifiable property.

---

## Criterion 2: Error cases are covered (invalid enum values, missing required fields, port conflicts)

**PASS**

REQ-6 (Section 4) explicitly covers:
- Invalid GAME_PRESET: exit with error listing valid presets (AC-6.1)
- Missing game files: exit with reinstall guidance (AC-6.2)
- Invalid GAMETYPE_* value: documented as server exits on startup (in variable description warning)
- Invalid MAP_* value: documented as game engine handles gracefully
- PORT out of range: prevented by panel validation rules (`min:1024|max:65535`)
- Xvfb failure: exit with error indicating virtual display failure

AC-3.7 and AC-3.8 also cover the error paths at the startup level. Port conflicts between multiple servers are inherently handled by OS/Wings port allocation -- not an egg-level concern.

---

## Criterion 3: Success metrics are measurable and verifiable

**PASS**

Section 6 defines 7 success metrics, each with a concrete target:
- "Clean start": observable done string output (100% after install)
- "No permission errors": zero occurrences of specific string in output
- "Port listening": UDP port response within 60s (measurable with netcat/nmap)
- "Player connect": manual UAT verification
- "Install success": 100% success rate on fresh allocation
- "TrueNAS SCALE": verified on specific instance
- "No Panel Mounts": grep-verifiable (no /rvs references)

All are either automatable or have clear manual verification steps.

---

## Criterion 4: The PRD is testable end-to-end (can you write a test plan from this PRD?)

**PASS**

A complete test plan is derivable:
1. **Install test**: Run install script on fresh allocation, verify AC-2.1 through AC-2.7
2. **Clean start test**: Start server, verify done string appears (AC-3.2)
3. **Port test**: Verify UDP ports PORT, PORT+1000, PORT+2000 are listening (AC-3.3 through AC-3.5)
4. **Variable test**: Verify all 20 variables present with correct rules (AC-4.1 through AC-4.8)
5. **Error case tests**: Invalid GAME_PRESET, missing files, Xvfb failure (AC-6.1 through AC-6.4)
6. **Idempotency test**: Re-run install, verify no corruption (AC-2.4)
7. **JSON validity test**: `jq .` passes (NFR-6)
8. **Metadata test**: Verify all AC-5.x fields
9. **Integration test**: Player connects via game client (Section 6)
10. **TrueNAS SCALE test**: Full flow on containerized Wings

---

## Criterion 5: Variable validation rules are complete and consistent (required vs nullable, max lengths)

**PASS**

The variable table in REQ-4 specifies all 20 variables with:
- Explicit `required` or `nullable` designation
- `max:64` or `max:128` length limits on all string fields
- `min:1024|max:65535` range on PORT
- `min:1|max:64` range on MAX_PLAYERS
- `in:` enum validation on GAME_PRESET, INTERNET_SERVER, INSTALL_OPENRVS
- Consistent `field_type: text` across all variables
- AC-4.6 explicitly requires rules match the table exactly

---

## Criterion 6: MAP slot 0 uses required; slots 1-5 use nullable

**PASS**

Section 4, REQ-4 variable table:
- Row 5: MAP_0 -- rules: `required|string|max:64`
- Rows 7, 9, 11, 13, 15: MAP_1 through MAP_5 -- rules: `nullable|string|max:64`

Additionally, "Variable Description Requirements" explicitly states:
> "MAP_0/GAMETYPE_0: Use `required` in rules"
> "MAP_1-5/GAMETYPE_1-5: Use `nullable` in rules"

AC-4.2 explicitly validates: "MAP_0 and GAMETYPE_0 use `required` in rules; slots 1-5 use `nullable`"

---

## Criterion 7: GAMETYPE slot 0 uses required; slots 1-5 use nullable

**PASS**

Section 4, REQ-4 variable table:
- Row 6: GAMETYPE_0 -- rules: `required|string|max:128`
- Rows 8, 10, 12, 14, 16: GAMETYPE_1 through GAMETYPE_5 -- rules: `nullable|string|max:128`

Same explicit statements and AC-4.2 coverage as MAP slots above.

---

## Criterion 8: GAMETYPE descriptions include warning about invalid values causing startup exit

**PASS**

Section 4 "Variable Description Requirements" explicitly states:
> "GAMETYPE_* descriptions: Must list all 6 valid game type class names with human-readable labels. Must include warning: 'An invalid game type will cause the server to exit on startup with an error.'"

AC-4.3 explicitly validates: "All GAMETYPE_* descriptions include the invalid-value startup-exit warning"

---

## Criterion 9: No ambiguous requirements that could be interpreted multiple ways

**PASS**

The PRD is notably precise:
- Exact image URIs specified (not "a Wine image")
- Exact done string specified (not "a success message")
- Exact port formula documented with defaults
- Exact variable rules in table form (not prose)
- Exact file paths (`/home/container`, `/mnt/server`)
- Valid enum values exhaustively listed (7 presets, 18 maps, 6 game types)
- GAME_PRESET behavior explicitly calls out that the Architect must document whether it overrides or defaults MAP/GAMETYPE values (Section 9, item 5)
- Error message text specified where relevant (AC-6.2 quotes the expected message)

One minor note: The relationship between GAME_PRESET and MAP_*/GAMETYPE_* variables (override vs. default) is deferred to Architect analysis (Section 9, item 5). This is acceptable because it depends on upstream implementation discovery, and the PRD explicitly flags it as requiring documentation.

---

## Summary

| # | Criterion | Verdict |
|---|-----------|---------|
| 1 | Testable acceptance criteria | PASS |
| 2 | Error cases covered | PASS |
| 3 | Measurable success metrics | PASS |
| 4 | End-to-end testability | PASS |
| 5 | Variable validation complete | PASS |
| 6 | MAP slot 0 required, 1-5 nullable | PASS |
| 7 | GAMETYPE slot 0 required, 1-5 nullable | PASS |
| 8 | GAMETYPE invalid-value warning | PASS |
| 9 | No ambiguous requirements | PASS |

---

## Verdict: DONE

**All 9 criteria pass.** PRD v2.0 addresses the prior review's blocking failure (missing error-case ACs) with comprehensive coverage in REQ-6 and AC-3.7/AC-3.8. No blocking or warning issues remain.
