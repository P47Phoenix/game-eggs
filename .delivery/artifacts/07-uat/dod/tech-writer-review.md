# DoD Validation — Tech Writer Review

**Validator role:** Technical Writer / UAT Stage 7 DoD Validator
**Date:** 2026-04-11
**Artifacts reviewed:**
- `.delivery/artifacts/07-uat/tech-writer/release-notes.md`
- `rainbow_six_3_raven_shield/README.md`
- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Gate Criteria Evaluation

### Gate 1 — Release notes are clear and accurate

**Result: PASS**

The release notes are structured as six numbered change sections, each covering one cohesive change. All six changes are verifiable against the actual artifacts:

- Rotation expansion to 6 slots: confirmed — MAP_3 through MAP_5 and GAMETYPE_3 through GAMETYPE_5 are present in the JSON with blank defaults.
- Inline valid-value lists in slots 1–5: confirmed — all MAP_1–MAP_5 and GAMETYPE_1–GAMETYPE_5 descriptions in the JSON contain the full inline lists without any cross-reference to slot 0.
- max:128 standardization for GAMETYPE_*: confirmed — all six GAMETYPE variables carry `max:128` in their rules.
- Startup failure warning in all GAMETYPE_* descriptions: confirmed — the warning "An invalid game type will cause the server to exit on startup with an error." appears in every GAMETYPE_0 through GAMETYPE_5 description.
- Operator warning in egg description: confirmed — the description field opens with the IMPORTANT warning block before the game description text.
- Known Issues section in README: confirmed — the section exists at line 118 of the README with Symptoms, Root cause, and Fix sub-sections.

The "What Is Not Changed" section accurately reflects the JSON (startup command, image reference, install script, config blocks, stop signal are identical to expected values; slot 0 descriptions are unchanged). No inaccuracies were found.

---

### Gate 2 — README Known Issues section is understandable to a non-expert operator

**Result: PASS**

The Known Issues / Operator Requirements section (README lines 118–131) uses a consistent problem-solving structure: Symptoms, Root cause, Fix. Each sub-section uses plain language:

- **Symptoms** describes observable behavior: crash on startup, exit code 1, "permission denied" in output. A non-expert can match this to what they actually see.
- **Root cause** is technical but self-contained. It explains *why* without requiring prior knowledge of Wings internals — the key fact (Wings puts data at /home/container, image expects /rvs) is stated explicitly.
- **Fix** names the specific action: configure the Wings node to bind-mount the server data directory to /rvs. The scope is clearly stated: "at the Wings server level, not in the Pterodactyl panel egg configuration."
- The NOTE callout correctly places responsibility on the third-party image, preventing operators from filing false bug reports against the egg.

One minor gap: the Fix section does not provide a link to Wings documentation or a specific configuration key name for the bind-mount. A non-expert will know *what* to do but may not know *where in Wings* to do it. This falls short of hand-holding but meets the "understandable and actionable" bar for an operator who manages a Wings node.

No rewrite required. Minor improvement opportunity: consider adding a link to the Wings server configuration documentation in a future revision.

---

### Gate 3 — Egg description operator warning is prominent and actionable

**Result: PASS**

The egg `description` field opens with the warning text before any other content:

> IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions.

This satisfies all three sub-criteria for an operator warning:

- **Prominent:** The word "IMPORTANT:" appears as the first word. The description field is visible when an operator selects the egg in the panel before creating a server, making this the earliest feasible notification point in the setup flow.
- **Actionable:** The warning specifies the required action (bind-mount server data directory to /rvs), names the default incorrect path (/home/container), and names the correct path (/rvs). An operator reading this knows what to do before they attempt to start a server.
- **Scoped:** The warning correctly states this is a Wings-level configuration change, not a panel egg setting, preventing operators from looking in the wrong place.

The release notes correctly characterize this as documentation of the workaround, not a fix, and that characterization is accurate.

---

### Gate 4 — No broken cross-references in variable descriptions ("See MAP_0" style)

**Result: PASS**

All variable descriptions in the egg JSON were inspected for cross-reference patterns. No "See MAP_0", "see slot 0", or similar forward/backward references appear in any variable description. Each MAP_* and GAMETYPE_* description (slots 1–5) contains the complete inline valid-value list, independent of any other variable.

The README environment variable table uses "See map list below" and "See game type list below" in the Allowed Values column for MAP_* and GAMETYPE_* variables. These are intra-document anchor references, not cross-variable references. Both referenced sections (Valid Map Names at line 81, Valid Game Types at line 105) exist and are complete. These resolve correctly within the README and are not broken.

The memory lesson applied: PTDL_v2 has no dropdown field type, so valid values must be in description text. This requirement is satisfied — all MAP and GAMETYPE variables carry inline value lists in their description fields.

---

## Summary

| Gate | Criterion | Result |
|------|-----------|--------|
| 1 | Release notes are clear and accurate | **PASS** |
| 2 | README Known Issues section is understandable to a non-expert operator | **PASS** |
| 3 | Egg description operator warning is prominent and actionable | **PASS** |
| 4 | No broken cross-references in variable descriptions | **PASS** |

**Overall verdict: ALL GATES PASS**

No blocking defects. One non-blocking improvement opportunity noted in Gate 2 (link to Wings bind-mount documentation). Artifacts are ready to proceed past Stage 7 UAT tech-writer gate.
