# Architect DoD Review — Stage 1 Idea Brief
## Rainbow Six 3: Raven Shield — Greenfield Egg (Scratch Build)

**Reviewer:** Architect Agent
**Date:** 2026-04-12
**Pipeline run:** run-2026-04-12-r6egg
**Input artifact:** `.delivery/artifacts/01-idea/po/idea-brief.md`
**Target egg:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`

---

## Gate Criterion Verdicts

### CRITERION 1 — The idea is technically feasible with the stated constraints

**PASS**

The approach is technically feasible. The plan is to use a standard Wine yolk image (`ghcr.io/ptero-eggs/yolks:wine_latest`) and write egg install/startup scripts that place everything in `/home/container`. This is a well-established pattern in this repository:

- **Mordhau Wine** (`mordhau/egg-mordhau-wine.json`) uses `ghcr.io/ptero-eggs/yolks:wine_latest` with `xvfb-run --auto-servernum wine ...` as its startup command.
- **Wine Generic** (`wine/generic/egg-wine-generic.json`) demonstrates the same pattern with install scripts that download and unpack game files to `/mnt/server` (which becomes `/home/container` at runtime).
- At least 38 eggs in this repository use Wine-based images, and 16 use Xvfb directly.

The upstream Dockerfile for `danpowell88/ravenshield_dedicatedserver` uses Ubuntu + Wine + Xvfb — the exact same runtime that the standard ptero-eggs Wine yolks provide. The install script can replicate the upstream Dockerfile's `RUN` steps (downloading from archive.org, extracting, patching OpenRVS) and the startup command can replicate `entrypoint.sh` logic using egg variables and `xvfb-run wine`.

All of this fits cleanly within the PTDL_v2 egg JSON format with no custom Docker image build required.

---

### CRITERION 2 — No obvious technical blockers are unaddressed

**PASS**

Potential blockers and their mitigations:

1. **Wine yolk includes Xvfb:** Confirmed. The Mordhau egg (`mordhau/egg-mordhau-wine.json`) uses `xvfb-run` directly in its startup command with the same `ghcr.io/ptero-eggs/yolks:wine_latest` image. No additional package installation is needed at runtime.

2. **Archive.org download reliability:** The brief acknowledges game files come from archive.org. This is an external dependency that could break, but it is the same source the upstream image uses. The install script runs once (not on every boot), so transient failures are manageable. The reference material confirms the URL is known from prior pipeline work.

3. **Non-root UID execution:** Pterodactyl runs containers as a non-root UID. Since all game data will be written to `/home/container` (which Pterodactyl bind-mounts with correct ownership), there is no permission conflict. The upstream image's `/rvs` VOLUME problem is eliminated entirely by this approach.

4. **Wine prefix initialization:** Wine creates a `.wine` prefix on first run. The Mordhau egg handles this by creating the directory structure in the install script. The same pattern applies here.

5. **Config file generation:** The upstream `entrypoint.sh` generates INI config files from environment variables before launching. This logic must be replicated in the egg's startup command (likely a bash one-liner or inline script). PTDL_v2 supports multi-line startup commands via `bash -c '...'` patterns, which is how other complex eggs handle runtime config generation.

No unaddressed technical blockers remain.

---

### CRITERION 3 — The scope is achievable (not too broad, not too narrow)

**PASS**

The scope is well-sized for a single egg deliverable:

**Included (appropriate):**
- Analyze upstream Dockerfile and entrypoint.sh (research phase)
- Select base image (already identified: Wine yolk)
- Write install script (download + extract + optional OpenRVS patch)
- Write startup command (config generation + Wine launch)
- Create full egg JSON with all variables
- Create README
- Test on TrueNAS SCALE

**Excluded (appropriate):**
- No upstream image modifications
- No upstream contributions
- No mod support beyond OpenRVS
- No Panel Mount documentation

The scope is a single egg JSON file plus a README — the standard unit of work in this repository. It is neither too broad (it is one game, one egg) nor too narrow (it covers the full lifecycle: install, configure, start).

---

### CRITERION 4 — Constraints are realistic and complete

**PASS**

All stated constraints are realistic:

| Constraint | Assessment |
|---|---|
| Must use PTDL_v2 egg JSON format | Standard. All eggs in the repo use this. |
| Must NOT require Panel Mounts or Wings `allowed_mounts` | Achievable — data lives in `/home/container`, no external mounts needed. |
| Must NOT require a custom-built Docker image | Achievable — standard Wine yolk provides Wine + Xvfb. |
| Game data must live in `/home/container` | Standard Pterodactyl pattern. Install script writes to `/mnt/server` which maps to `/home/container`. |
| Must work on containerized Wings (TrueNAS SCALE) | Achievable — by avoiding Panel Mounts and using standard images, there are no Wings-specific dependencies. |

**Completeness check — no missing constraints identified:**
- The memory lesson about `docker_images` key format (key must equal value URI) is not explicitly stated as a constraint, but this is a repo-wide convention that the developer will follow. Not a brief-level omission.
- No constraint on maximum egg variable count is needed (PTDL_v2 has no hard cap).

---

### CRITERION 5 — The reference material is sufficient for downstream architecture work

**PASS**

The brief provides:

1. **Upstream repo reference** (`github.com/danpowell88/ravenshield_dedicatedserver`) — contains the Dockerfile and entrypoint.sh that document the full install and launch process. This is the primary technical reference.

2. **Validated reference data from prior pipeline:**
   - 18 valid map names (enumerated)
   - 6 valid game types with full class paths (enumerated)
   - Port formula: PORT, PORT+1000, PORT+2000
   - Done string: "OpenRVS is up to date"
   - Variable loop range: MAP_0–MAP_31, GAMETYPE_0–GAMETYPE_31
   - Download sources: archive.org for game files, GitHub releases for OpenRVS

3. **Existing repo patterns** — 38+ Wine eggs and the generic Wine egg provide templates for install scripts, startup commands, and variable definitions.

This is sufficient for an architect to produce a full technical design without additional research. The entrypoint.sh and Dockerfile contain all implementation details needed (env vars, file paths, launch arguments, config file format).

---

### CRITERION 6 (KEY RISK) — Is there a suitable public Docker image with Wine + Xvfb that can be used as a base?

**PASS**

**Confirmed: `ghcr.io/ptero-eggs/yolks:wine_latest`** is the standard Pterodactyl Wine image that includes both Wine and Xvfb.

Evidence:
- `mordhau/egg-mordhau-wine.json` uses this exact image with `xvfb-run --auto-servernum wine ...` as its startup command.
- At least 16 eggs in this repository use `xvfb-run` with Wine yolk images.
- This is a first-party Pterodactyl community image (hosted at `ghcr.io/ptero-eggs/yolks`), meaning it is maintained, publicly available, and does not require custom builds.
- The image is already used by production eggs in this repository, confirming it works with Pterodactyl's non-root UID execution model.

No custom Docker image is needed. The key risk is fully mitigated.

---

## Summary Table

| # | Criterion | Verdict |
|---|-----------|---------|
| 1 | Technically feasible with stated constraints | **PASS** |
| 2 | No obvious technical blockers unaddressed | **PASS** |
| 3 | Scope is achievable | **PASS** |
| 4 | Constraints are realistic and complete | **PASS** |
| 5 | Reference material sufficient for architecture | **PASS** |
| 6 | Key risk: suitable public Docker image exists | **PASS** |

**Overall gate result: DONE — 6 of 6 criteria PASS**

---

## Disposition

The idea brief is approved to proceed to Stage 2 (Refine / PRD). The following notes apply to downstream work:

1. **Base image decision is locked:** Use `ghcr.io/ptero-eggs/yolks:wine_latest` as the runtime Docker image and `ghcr.io/ptero-eggs/installers:debian` as the install container. This follows the established pattern from Mordhau, Wine Generic, and dozens of other eggs.

2. **Memory lesson — `docker_images` key format:** The egg JSON must use `"ghcr.io/ptero-eggs/yolks:wine_latest": "ghcr.io/ptero-eggs/yolks:wine_latest"` (key equals value equals full URI). Do not use a human-readable label as the key.

3. **Memory lesson — read upstream entrypoint.sh before writing PRD:** The architecture phase must read the actual `entrypoint.sh` from `github.com/danpowell88/ravenshield_dedicatedserver` to confirm env var names, port formula, done string detection, and map/gametype handling. The brief's reference data section provides a summary, but the source must be verified.

4. **Startup command complexity:** The upstream entrypoint.sh generates INI config files dynamically from environment variables using a loop (MAP_0–MAP_31, GAMETYPE_0–GAMETYPE_31). The architecture must decide whether to replicate this as an inline `bash -c '...'` startup command or as a separate approach. This is a design decision for Stage 3.

5. **Repo PR checklist note:** This egg uses a standard yolk image, so no custom Docker image justification is required in the PR.
