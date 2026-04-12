# UAT Test Plan — Rainbow Six 3: Raven Shield Egg Changes

**Date:** 2026-04-11
**Author:** QA Engineer
**Pipeline:** run-2026-04-11-r6fix
**Active Track:** Track A (static JSON) + Track B2 (documentation)
**Target files:**
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
- `rainbow_six_3_raven_shield/README.md`

---

## 1. Purpose

This UAT test plan validates the completed changes to the Rainbow Six 3: Raven Shield Pterodactyl egg. The changes under test are:

1. Expansion of map and game type rotation slots from 2 to 6 (MAP_0–MAP_5, GAMETYPE_0–GAMETYPE_5)
2. Description content: full inline map list and game type list in every slot (no cross-references)
3. Rules normalisation: all GAMETYPE_* fields use `max:128`
4. Required/nullable enforcement: slot 0 required, slots 1–5 nullable
5. Error-case documentation: invalid GAMETYPE value warning present in all six GAMETYPE descriptions
6. docker_images key-equals-value URI correctness
7. Egg description /rvs operator warning at the top of the description field
8. README Known Issues section documenting the root cause and Wings bind-mount workaround

---

## 2. Track Selection

| Track | Condition | Status |
|-------|-----------|--------|
| Track A — Static JSON | Always active | **ACTIVE** |
| Track B2 — Documentation | Gate 1 = no image-level override env var for /rvs | **ACTIVE** |
| Track B1 — Override Variable | Gate 1 = image exposes override env var | **NOT ACTIVE** |
| Empirical Docker tests | Docker available locally | **DEFERRED** (document at sign-off) |

Track B2 is active because the Docker image (`ghcr.io/danpowell88/ravenshield_dedicatedserver`) hardcodes `/rvs` with no env-var override, and the fix is therefore documentation-only (egg description warning + README Known Issues section). The empirical Docker test (confirming the `--user 1000:1000` failure) is expected to remain a known issue and is deferred; the static description checks are mandatory.

---

## 3. Test Scope

### 3.1 In Scope

**Track A — Static JSON (all mandatory):**

| # | Scope Area |
|---|------------|
| A-01 | JSON syntactic validity |
| A-02 | `meta.version` equals `"PTDL_v2"` |
| A-03 | `docker_images` key equals value equals full URI |
| A-04 | Egg `description` starts with /rvs operator warning |
| A-05 | 6 MAP slots present (MAP_0–MAP_5) |
| A-06 | 6 GAMETYPE slots present (GAMETYPE_0–GAMETYPE_5) |
| A-07 | MAP_0 and GAMETYPE_0 rules begin with `required` |
| A-08 | MAP_1–MAP_5 and GAMETYPE_1–GAMETYPE_5 rules begin with `nullable` |
| A-09 | All GAMETYPE_* rules contain `max:128`; none contain `max:64` |
| A-10 | MAP_1–MAP_5 descriptions each contain all 18 valid map names inline |
| A-11 | GAMETYPE_1–GAMETYPE_5 descriptions each contain all 6 game type values with labels |
| A-12 | All 6 GAMETYPE_* descriptions include invalid-value startup-exit warning |
| A-13 | New slots (MAP_3–5, GAMETYPE_3–5) have empty default_value, user_viewable true, user_editable true, field_type text |
| A-14 | Unmodified variables regression check |

**Track B2 — Documentation (all mandatory):**

| # | Scope Area |
|---|------------|
| B2-01 | Egg description contains /rvs warning with root cause and Wings workaround |
| B2-02 | Egg description does not claim the crash is fixed |
| B2-03 | README contains Known Issues section with root cause and Wings workaround |

**Empirical — Docker (deferred):**

| # | Scope Area | Status |
|---|------------|--------|
| E-01 | `docker run --user 1000:1000 ghcr.io/danpowell88/ravenshield_dedicatedserver` fails with permission denied (confirms B2 is the correct track) | DEFERRED |

### 3.2 Out of Scope

- Building or modifying the Docker image
- Live Pterodactyl panel import tests
- Network and port reachability testing (UDP 7777 / 8777 / 9777)
- Entrypoint script internal logic beyond observable stdout/exit code
- Other egg files in the repository
- CI/CD pipeline configuration

---

## 4. Test Environment Requirements

### 4.1 Track A and Track B2 (Static)

| Requirement | Detail |
|-------------|--------|
| File access | `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` readable |
| File access | `rainbow_six_3_raven_shield/README.md` readable |
| JSON parser | Python 3 (`python -m json.tool`) or any JSON-compliant parser |
| No network access required | All tests are file-inspection only |
| No Docker required | |
| No Pterodactyl panel required | |

### 4.2 Empirical Docker Tests (Deferred)

| Requirement | Detail |
|-------------|--------|
| Docker Engine | Installed and running |
| Network access | Pull from `ghcr.io` |
| Image | `ghcr.io/danpowell88/ravenshield_dedicatedserver` (latest) |

---

## 5. Test Cases Summary

| TC ID | Name | Track | Method | Priority |
|-------|------|-------|--------|----------|
| TC-A-01 | JSON Syntactic Validity | A | Static | Critical |
| TC-A-02 | PTDL_v2 Meta Version | A | Static | High |
| TC-A-03 | docker_images Key Equals Value | A | Static | High |
| TC-A-04 | Egg Description /rvs Warning Placement | A | Static | High |
| TC-A-05 | 6 MAP Slots Present | A | Static | Critical |
| TC-A-06 | 6 GAMETYPE Slots Present | A | Static | Critical |
| TC-A-07 | MAP_0 and GAMETYPE_0 Required Rules | A | Static | Critical |
| TC-A-08 | MAP_1–5 and GAMETYPE_1–5 Nullable Rules | A | Static | Critical |
| TC-A-09 | All GAMETYPE_* Rules Contain max:128 | A | Static | High |
| TC-A-10 | MAP_1–5 Descriptions Contain All 18 Map Names | A | Static | High |
| TC-A-11 | GAMETYPE_1–5 Descriptions Contain All 6 Values with Labels | A | Static | High |
| TC-A-12 | All 6 GAMETYPE Descriptions Have Invalid-Value Warning | A | Static | High |
| TC-A-13 | New Slot Field Values (MAP_3–5, GAMETYPE_3–5) | A | Static | High |
| TC-A-14 | Unmodified Variables Regression Check | A | Static | Medium |
| TC-B2-01 | Egg Description /rvs Warning Content Completeness | B2 | Static | High |
| TC-B2-02 | Egg Description No Claim of Fix | B2 | Static | Medium |
| TC-B2-03 | README Known Issues Section Content | B2 | Static | High |
| TC-E-01 | Docker --user 1000:1000 Fails With Permission Denied | Empirical | Docker | Low (deferred) |

---

## 6. Validation Matrix (AC Coverage)

| Story AC | Description (abbreviated) | Test Case(s) | Track |
|----------|---------------------------|--------------|-------|
| AC-A-01 to AC-A-05 | MAP_1–MAP_5 descriptions contain full inline map list | TC-A-10 | A |
| AC-A-06 | MAP_0 description unchanged (regression) | TC-A-14 | A |
| AC-A-07 to AC-A-11 | GAMETYPE_1–GAMETYPE_5 descriptions contain all 6 types with labels | TC-A-11 | A |
| AC-A-12 | GAMETYPE_0 description unchanged (regression) | TC-A-14 | A |
| AC-A-13 to AC-A-18 | All GAMETYPE rules contain max:128 | TC-A-09 | A |
| AC-A-19 | MAP_0 rules begins with required | TC-A-07 | A |
| AC-A-20 | GAMETYPE_0 rules begins with required | TC-A-07 | A |
| AC-A-21 | MAP_1–MAP_5 rules begin with nullable | TC-A-08 | A |
| AC-A-22 | GAMETYPE_1–GAMETYPE_5 rules begin with nullable | TC-A-08 | A |
| AC-A-23 | MAP_3, MAP_4, MAP_5 present | TC-A-05 | A |
| AC-A-24 | GAMETYPE_3, GAMETYPE_4, GAMETYPE_5 present | TC-A-06 | A |
| AC-A-25 | New map slots have empty default and nullable rules | TC-A-13 | A |
| AC-A-26 | New gametype slots have empty default and nullable rules | TC-A-13 | A |
| AC-A-27 | New slots have user_viewable and user_editable true | TC-A-13 | A |
| AC-A-28 | New slots have field_type text | TC-A-13 | A |
| AC-A-29 (static) | GAMETYPE descriptions warn invalid value causes exit | TC-A-12 | A |
| AC-A-29 (empirical) | Container exits non-zero on invalid GAMETYPE | TC-E-01 | Empirical |
| AC-A-30 | JSON valid (python -m json.tool exits 0) | TC-A-01 | A |
| AC-A-31 | meta.version is PTDL_v2 | TC-A-02 | A |
| AC-A-32 | docker_images key equals value equals full URI | TC-A-03 | A |
| AC-A-33 | Unmodified variables byte-for-byte identical | TC-A-14 | A |
| AC-B2-01 | Egg description contains crash warning with root cause | TC-B2-01, TC-A-04 | B2 |
| AC-B2-02 | Warning appears at top of description | TC-A-04 | A/B2 |
| AC-B2-03 | Description does not claim crash is fixed | TC-B2-02 | B2 |
| AC-B2-04 | README has Known Issue section with cause and workaround | TC-B2-03 | B2 |

---

## 7. Entry and Exit Criteria

### 7.1 Entry Criteria

- Final egg JSON file is available at `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
- Final README is available at `rainbow_six_3_raven_shield/README.md`
- QA engineer has confirmed Track B2 is active (no image-level /rvs override)
- Python 3 is available for JSON validity check

### 7.2 Exit Criteria (Definition of Done)

All of the following must be satisfied to close UAT:

**Track A (all required):**
- [ ] TC-A-01 through TC-A-14 all pass

**Track B2 (all required):**
- [ ] TC-B2-01 through TC-B2-03 all pass

**Empirical (deferred):**
- [ ] TC-E-01 deferred with documented reason, OR passes confirming expected failure

Any TC-A or TC-B2 failure blocks sign-off. A deferred TC-E-01 does not block sign-off provided the reason is documented.

---

## 8. Known Gaps and Risks

| Gap / Risk | Mitigation |
|------------|------------|
| Docker not available locally | TC-E-01 deferred; static warning checks (TC-A-12, TC-B2-01) remain mandatory and provide coverage of the user-visible requirement |
| No pre-change baseline snapshot committed | TC-A-14 regression check relies on tester having access to the unmodified original; developer must supply baseline if not in git history |
| External Docker image availability | Noted out of scope; image availability is an operational dependency, not a defect in this egg |
| MAP_* and GAMETYPE_* fields accept any string (no panel validation) | Documented in README Known Limitations; not testable via static JSON; operator education approach accepted |

---

*End of UAT Test Plan*
