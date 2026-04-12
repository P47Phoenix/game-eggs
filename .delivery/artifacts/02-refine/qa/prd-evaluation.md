# PRD Evaluation: Tom Clancy's Rainbow Six 3: Raven Shield Dedicated Server Egg

**Evaluator:** QA Engineer (Evaluator role)
**PRD Version:** 1.0
**Evaluation Date:** 2026-04-11
**PRD Location:** `.delivery/artifacts/02-refine/po/prd.md`

---

## 1. Testability Assessment

### Well-Specified (Testable As-Is)

| AC / FR | Assessment |
|---|---|
| AC-1 (JSON validity) | Fully testable. Command is exact, exit code criterion is clear. |
| AC-5 (variable array inspection) | Testable via JSON path queries. All 10 variable names are enumerated and rules strings are explicit. |
| AC-6 (author field) | Trivially testable. Expected value is an exact string. |
| AC-8 (root README row) | Testable by diff inspection and alphabetical-order check. |
| AC-9 (docker_images field) | Testable via JSON path query. Exact key and value are specified. |
| FR-2 (file count) | Testable: list directory, assert exactly 2 files. |
| FR-12 / FR-13 (field constraints) | Testable by iterating the variables array. |
| NFR-1 (JSON validity) | Covered by AC-1. |
| NFR-5 (encoding / line endings) | Testable via `file` command or hex dump. |

### Conditionally Testable (Requires Environment)

| AC / FR | Gap |
|---|---|
| AC-2 (panel import) | Requires a live Pterodactyl v1.x instance. No specification of which v1.x minor version(s) to target, or whether a specific wings version is required. Testable once environment is defined, but the environment is not specified. |
| AC-3 (startup direct invocation) | "Panel executes the startup command directly" — verifiable by reading the panel's process table or startup log, but the PRD does not specify what evidence (log line, process name, etc.) constitutes proof. Weakly testable. |
| AC-4 (ports open) | "Accepting traffic" is underspecified. A port can be open (bound) without responding meaningfully to arbitrary UDP. The PRD should clarify: bound and listening is sufficient, or must respond to a Raven Shield query packet? |
| AC-10 (server boots) | "Logs indicate a successful initialisation" — the expected log string is not defined in the AC itself (it is deferred to OQ-2 / FR-10). This AC cannot be written into a repeatable test case until OQ-2 is resolved. |

### Too Vague to Test Directly

| AC / FR | Issue |
|---|---|
| AC-7 (README review) | "Reviewed" is subjective. The five required sections are enumerated, but no minimum content criteria are given (e.g., does the installation section need to cover both panel UI import and CLI import? How many steps?). A checklist exists but its pass/fail line is undefined. |
| AC-11 (PR checklist pass) | References "the repository's existing PR checklist" without quoting or citing the checklist. A tester needs to locate that checklist independently. Low risk (checklist presumably exists in the repo), but the AC should link to it. |
| NFR-2 (panel compatibility) | "No errors or warnings" — Pterodactyl panel v1.x may emit informational notices that are not errors. The threshold is unclear. |
| FR-10 (config.startup.done string) | Explicitly deferred to runtime observation (OQ-2). The requirement is present but untestable until the research action is completed and its output is incorporated into the PRD. |

---

## 2. Completeness Gaps

**GAP-1: `GAME_PRESET` variable has no mechanical effect defined.**
FR-11 defines `GAME_PRESET` as controlling "overall game mode preset," but no requirement states how the egg actually uses this variable. The startup command (FR-9) references `GAMETYPE_0`, not `GAME_PRESET`. There is no `config.files` templating requirement that maps `GAME_PRESET` to a config file value. The variable is user-visible but its mechanism is unspecified. If it is purely advisory (a user reminder to set GAMETYPE_* correctly), the PRD should say so explicitly. If it drives something, that mapping is missing.

**GAP-2: `config.files` parser requirements are absent.**
The PRD mentions `config.files` as a sub-field of `config` (FR-4) but never specifies its contents. For Unreal Engine 2 games, server configuration typically lives in `.ini` files. If `SERVER_NAME`, `MAX_PLAYERS`, or other variables must be written into `RavenShield.ini` (or `RavenShield.ServerSettings.ini`), those `config.files` rules are required. Without them, panel environment variables may not reach the server process at all — yet no requirement addresses this gap.

**GAP-3: No requirement covers `config.logs`.**
FR-4 lists `config.logs` as a required sub-field but no FR specifies its value. Other eggs in this repository typically set `config.logs` to `{}` or a path. The implementer has no specification to follow.

**GAP-4: Variable `MAP_1` default value `Training_1` appears incorrect.**
`Training_1` is a single-player training map, not a standard multiplayer map. This may cause server startup failure when the map rotation is used with the default values. The PRD should either use a known-valid MP map name or flag this as "TBD pending OQ research."

**GAP-5: No requirement for port allocation in the egg's `allocations` or `config` section.**
US-4 states "all three required UDP ports (7777, 8777, 9777) declared in the egg." FR-3.5 documents the ports, but no FR specifies exactly where in the PTDL_v2 JSON these ports must appear (e.g., is there a `ports` or `allocations` array?). Pterodactyl eggs declare ports in the startup command and/or as environment variables; the PRD does not require a specific JSON representation for ports 8777 and 9777 beyond "documented in README."

**GAP-6: No minimum Pterodactyl panel version specified beyond "v1.x".**
AC-2 and NFR-2 say "v1.x" but the PTDL_v2 schema was introduced at a specific minor version. Testing against an early v1.x panel may produce false negatives. The minimum compatible minor version should be specified.

**GAP-7: Installation script behaviour is underspecified.**
FR-3.6 allows a no-op script if the image is self-contained but also says "performs any setup steps required." Since the required setup steps depend on OQ-1 and OQ-4 (image inspection), the installation script requirement cannot be fully written until those questions are resolved. The PRD should explicitly note this dependency.

**GAP-8: No requirement for `field_type` on the `GAME_PRESET` variable.**
FR-13 mandates `"text"` for all variables. Given that `GAME_PRESET` is constrained to an enum (`in:COOP,ADVERSARIAL`), there is a user-experience argument for a different field type. The PRD explicitly locks all types to `"text"`, which is consistent with repo practice, but no AC validates `field_type` values individually — only FR-13 states the rule.

---

## 3. Ambiguities

**AMB-1: "Direct binary invocation" (FR-6) vs. Wine invocation.**
FR-6 prohibits shell script wrappers but FR-9 lists `wine /rvs/System/UCC.exe ...` as a valid startup form. It is ambiguous whether invoking `wine` counts as a "direct binary invocation." The intent appears to be "no `.sh` wrapper," but the wording of FR-6 could be read as prohibiting `wine` (wine is an intermediary process). Clarify that `wine <binary>` satisfies FR-6.

**AMB-2: "Alphabetical order" for the root README row (FR-20 / AC-8).**
Does "alphabetical order" apply to the full game title "Tom Clancy's Rainbow Six 3: Raven Shield" or a shortened form? "Tom Clancy's..." sorts to T, while "Rainbow Six" sorts to R, and "Raven Shield" also to R. The existing table style (referenced but not quoted) governs this, but the PRD does not cite which style is authoritative.

**AMB-3: `nullable` rule and `default_value` for MAP_1, MAP_2, GAMETYPE_1, GAMETYPE_2.**
FR-11 assigns non-empty default values to all four nullable variables (e.g., MAP_1 defaults to `Training_1`). A `nullable` rule normally means the field may be empty, but providing a non-empty default means the field is never actually empty by default. The PRD is internally consistent but the intent is ambiguous: should users actively blank these fields to run a single-map rotation, or are the defaults always used? The description "Leave blank to use only MAP_0" implies blanking is valid, but the default is not blank.

**AMB-4: "Reliably indicates the server has finished starting" (FR-10).**
"Reliably" is a quality judgment. No tolerance for false positives or timing constraints is given. If the log line appears before the UDP port is actually bound, the panel may mark the server ready prematurely. The criterion for "reliable" needs a definition (e.g., "appears within N seconds of port bind" or "only emitted once per successful startup").

**AMB-5: GAME_PRESET values are `COOP` and `ADVERSARIAL` (uppercase), but the description refers to these as "preset" names.**
It is unclear whether these strings are passed to the server process, used to select a pre-baked config template, or serve only as a UI label for the operator. See also GAP-1.

---

## 4. Risk Flags

**RISK-1 (HIGH): All three high-priority open questions (OQ-1, OQ-2, OQ-3) block core deliverables.**
- OQ-1 blocks the startup command (FR-6 through FR-9, AC-3, AC-10).
- OQ-2 blocks the panel ready-state detection (FR-10, AC-10).
- OQ-3 may block repo inclusion entirely if the image requires a CD key.

The PRD correctly labels these as "RESEARCH REQUIRED" but planning cannot proceed to implementation until at least OQ-1 and OQ-2 are resolved. If the image is unavailable or requires licensing, the entire egg may be rejected at PR review. This is the single largest risk to the deliverable.

**RISK-2 (HIGH): External Docker image dependency with no SLA.**
The egg depends on `ghcr.io/danpowell88/ravenshield_dedicatedserver`, a community-maintained image with no uptime guarantee. If the image is deleted, rate-limited, or breaks its API, all servers using this egg stop working. The PRD acknowledges this in FR-19 but does not require a fallback strategy or a digest pin as a hard requirement. This is acceptable given the out-of-scope note, but testers should flag any test environment setup failure caused by image unavailability as an infrastructure issue, not an egg defect.

**RISK-3 (MEDIUM): `config.files` gap may cause silent misconfiguration.**
If `SERVER_NAME` and `MAX_PLAYERS` are not written into the server's `.ini` file at startup, the server will run with hardcoded defaults regardless of panel variable values. This risk is high-impact (user-facing misconfiguration) but silent (no error, server still starts). Without a `config.files` specification (GAP-2), this may not be caught until end-to-end testing.

**RISK-4 (MEDIUM): MAP_1 default value `Training_1` likely causes map-rotation errors.**
If the map rotation is parsed on startup and `Training_1` is not a valid multiplayer map identifier, the server may fail to start or skip the map silently. This would cause AC-10 to fail with default values. Low effort to mitigate: replace with a known valid MP map name or leave as empty string.

**RISK-5 (LOW): Wine-based startup command may require additional runtime flags.**
If the image is Wine-based, `wine` may require `WINEPREFIX`, `WINEARCH`, or display environment variables (`DISPLAY` or `XVFB`). These are not addressed in the PRD. If the image's `ENTRYPOINT` already handles this, the risk is moot, but it cannot be confirmed until OQ-1 is resolved.

**RISK-6 (LOW): PTDL_v2 schema is not formally published.**
The PRD references "PTDL_v2 schema" but no normative schema document (JSON Schema or otherwise) is cited. Conformance is validated by import into a live panel (AC-2), which requires a test environment. If the schema has undocumented required fields or forbidden field combinations, the egg may fail import in ways not caught by JSON linting alone.

---

## 5. Overall Verdict

**VERDICT: REVISE**

### Mandatory Before Planning Can Proceed

1. **Resolve OQ-1 and OQ-2** (or formally defer them with a blocker flag on the implementation tasks). The startup command and `config.startup.done` string are core to the egg's function and cannot be placeholder values in a shippable artifact.

2. **Specify `config.files` contents** (GAP-2) or explicitly state "no config file templating required" with a rationale. This is not a cosmetic gap — it determines whether panel variables reach the server process.

3. **Clarify GAME_PRESET mechanical effect** (GAP-1 / AMB-5). Either define how it maps to server configuration, or restate it as a documentation-only field with no server-side effect.

4. **Fix or flag MAP_1 default value** (GAP-4 / RISK-4). Replace `Training_1` with a valid multiplayer map name or document it as a known-invalid placeholder.

### Recommended Before Planning (Non-Blocking but High Value)

5. Specify `config.logs` contents (GAP-3).
6. Clarify Wine vs. direct invocation in FR-6 (AMB-1).
7. Define minimum Pterodactyl panel minor version (GAP-6).
8. Tighten AC-7 README review criteria (Section 1, "Too Vague" row).
9. Add a pointer to the PR checklist file in AC-11.

### What Is Well-Done

- The variable table (FR-11) is exceptionally detailed and directly testable.
- The file structure requirements (FR-1 through FR-3) are precise and testable.
- The out-of-scope section cleanly bounds the work.
- Open questions are honestly flagged rather than hidden.
- The success metrics table adds measurability that many PRDs omit.

The PRD is structurally sound and about 70% complete. The gaps are concentrated in the runtime-behaviour layer (startup command, config file wiring, ready-state detection) — exactly the area that determines whether the egg actually works. Resolving the four mandatory items above would make this PRD approvable.
