# QA DoD Validation — Round 2
**Date:** 2026-04-11
**Reviewer:** QA Engineer (DoD Validator)
**Artifact reviewed:** `.delivery/artifacts/02-refine/po/prd.md` (v1.1)
**Previous round blocking issue:** No AC for invalid GAME_PRESET / GAMETYPE values.

---

## Blocking Criteria Evaluation

### 1. Acceptance criteria cover the happy path (server starts successfully)
**PASS** — AC-10 explicitly covers the default-variable happy path: server starts with `GAME_PRESET=COOP`, `MAP_0=Airport`, `GAMETYPE_0=R6Game.R6TerroristHuntCoopGame`, `MAX_PLAYERS=16`, `NAME=Pterodactyl Raven Shield`; entrypoint downloads files, applies OpenRVS patch, console shows `OpenRVS is up to date`, server accepts connections.

---

### 2. Acceptance criteria cover at least one error/edge case (invalid GAME_PRESET, invalid GAMETYPE) ← previously FAILED
**PASS** — The PO has added two new ACs that directly address this:
- **AC-13**: Invalid `GAME_PRESET` (e.g. `GAME_PRESET=INVALID`) → entrypoint prints an error identifying the invalid preset and exits with code 1. Server must not start silently.
- **AC-14**: Invalid `GAMETYPE_*` (e.g. `GAMETYPE_0=R6Game.InvalidMode`) → entrypoint prints a warning identifying the invalid gametype and exits without launching the server process.

Both ACs are specific, observable, and testable. Previously blocking issue is resolved.

---

### 3. All 10+ env vars have defaults specified
**PASS** — 14 variables are defined across FR-9 (vars 1–10) and FR-10 (vars 11–14). Defaults are:
- GAME_PRESET: `COOP`
- PORT: `7777`
- NAME: `Pterodactyl Raven Shield`
- MAX_PLAYERS: `16`
- MAP_0: `Airport`
- MAP_1: `Bank`
- MAP_2: `Shipyard`
- GAMETYPE_0: `R6Game.R6TerroristHuntCoopGame`
- GAMETYPE_1: `R6Game.R6TerroristHuntCoopGame`
- GAMETYPE_2: `R6Game.R6TerroristHuntCoopGame`
- ADMIN_PASSWORD: `""` (empty, nullable — explicitly permitted by `nullable` rule)
- GAME_PASSWORD: `""` (empty, nullable — explicitly permitted by `nullable` rule)
- INTERNET_SERVER: `true`
- INSTALL_OPENRVS: `true`

All 14 have a specified default value (including empty string for optional nullable fields, which is valid per the PRD's own rule specification). AC-5 cross-checks that all 14 are present with non-empty defaults **or** explicitly `nullable` rules for optional fields — consistent with the variable definitions.

---

### 4. The "done" string for startup detection is specified and correct ("OpenRVS is up to date")
**PASS** — Specified in two places:
- FR-8: `config.startup` must contain a `done` key with value `OpenRVS is up to date`.
- AC-11: Verifies the `config.startup.done` field value is exactly `"OpenRVS is up to date"`.

Additionally AC-10 describes the happy-path end state as the console showing this exact string. The done string is unambiguous and consistent throughout the PRD.

---

### 5. Port requirements are testable (panel import exposes correct ports)
**PASS** — Section 3.5 declares all three UDP ports (7777, 8777, 9777) with their derivation. AC-4 is a testable criterion: "GIVEN the running server, WHEN checked from the host, THEN UDP ports 7777, 8777, and 9777 are all open and accepting traffic." AC-2 covers panel import exposing all 14 environment variables (implicitly including PORT). Success Metrics table includes "Port coverage: All 3 UDP ports documented and reachable — Manual netcat/nmap check against a running instance." The port ACs are concrete and verifiable.

---

## Summary Table

| Criterion | Result | Note |
|---|---|---|
| Happy path AC | PASS | AC-10 covers default-config server start through `OpenRVS is up to date` |
| Error/edge case AC (was blocking) | PASS | AC-13 (invalid GAME_PRESET) and AC-14 (invalid GAMETYPE) added in v1.1 |
| All 10+ env vars have defaults | PASS | All 14 vars defined; optional fields use `nullable` rule with empty-string default |
| Done string specified and correct | PASS | `"OpenRVS is up to date"` in FR-8, AC-10, and AC-11 |
| Port requirements testable | PASS | AC-4 specifies netcat/nmap verification; 3 UDP ports declared with derivation |

---

## Final Verdict

**DONE**

All five blocking criteria pass. The single blocking issue from Round 1 (missing ACs for invalid GAME_PRESET and GAMETYPE values) has been resolved by the addition of AC-13 and AC-14. No new blocking issues were identified. The PRD is sufficient to proceed to implementation.
