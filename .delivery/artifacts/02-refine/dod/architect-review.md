# Architect DoD Review -- Raven Shield Egg PRD v2.0 (Rebuilt)

**Reviewer:** Solution Architect (DoD Validator)
**Date:** 2026-04-12
**PRD:** `.delivery/artifacts/02-refine/po/prd.md` v2.0
**Verdict:** DONE

---

## Criterion 1: Technical approach is feasible (Wine yolk base image + egg install script)

**PASS**

The approach of using a ptero-eggs Wine yolk as the runtime image and a Debian installer container for the install script is the standard pattern used by every Wine-based egg in this repository (see `wine/generic/egg-wine-generic.json`). The install script deploys game files to `/mnt/server` (mapped to `/home/container` at runtime), and a `start.sh` script handles config generation and launch. This is a proven, well-established pattern across the repo.

---

## Criterion 2: Docker image selection is valid (must be a real, existing public image)

**PASS**

Verified via GitHub API (`gh api repos/ptero-eggs/yolks/contents/wine/latest/Dockerfile`) that `ghcr.io/ptero-eggs/yolks:wine_latest` exists in the `ptero-eggs/yolks` repository. The image is publicly available and actively maintained. Base image is `ghcr.io/ptero-eggs/yolks:debian_trixie`.

The PRD correctly specifies `docker_images` with key = value = URI:
```json
"docker_images": {
    "ghcr.io/ptero-eggs/yolks:wine_latest": "ghcr.io/ptero-eggs/yolks:wine_latest"
}
```

This satisfies the memory lesson requirement. Note: the PRD also mentions offering multiple options (Latest, Staging, Devel) following the wine generic pattern -- since the memory lesson mandates key=value=URI, the Developer should use only `wine_latest` unless there is a verified reason to offer alternatives.

---

## Criterion 3: Install script approach can replicate the upstream entrypoint's game setup

**PASS**

The PRD correctly identifies the required install steps: download game files from archive.org, extract to `/mnt/server`, optionally patch with OpenRVS from GitHub releases, and deploy a `start.sh` helper. This is a clean separation of install-time vs runtime concerns. Section 9 correctly delegates URL extraction to the Architect/Developer phase, which is appropriate since the URL may change.

Install container `ghcr.io/ptero-eggs/installers:debian` with entrypoint `bash` is the standard installer used across the repo.

---

## Criterion 4: Startup command approach is sound (Wine + Xvfb from /home/container)

**PASS** (with important clarification)

**Critical validation performed**: The Wine yolk Dockerfile at `ptero-eggs/yolks/wine/latest/Dockerfile` installs `xvfb` and `xauth` packages. The yolk's `entrypoint.sh` already starts Xvfb before evaluating `${STARTUP}`:

```bash
if [[ $XVFB == 1 ]]; then
    Xvfb :0 -screen 0 ${DISPLAY_WIDTH}x${DISPLAY_HEIGHT}x${DISPLAY_DEPTH} &
fi
# ... later ...
eval ${MODIFIED_STARTUP}
```

The ENV default is `XVFB=1` and `DISPLAY=:0`. Therefore, by the time `bash /home/container/start.sh` runs, Xvfb is ALREADY running on `:0`.

**Important clarification for Developer**: REQ-3 item 3 ("Start Xvfb on a virtual display") is REDUNDANT. The yolk entrypoint handles Xvfb automatically. The `start.sh` script should NOT start its own Xvfb instance -- doing so would conflict with the already-running instance. The script should simply use the pre-set `DISPLAY=:0` environment variable.

This is non-blocking because the Developer can trivially skip the Xvfb step in `start.sh`, but it is worth noting explicitly.

---

## Criterion 5: Port allocation requirements are correct (PORT, PORT+1000, PORT+2000)

**PASS**

The three-port formula (PORT for game, PORT+1000 for beacon, PORT+2000 for server) is correctly documented. The description field includes the operator warning about needing three ports allocated. The PORT variable has proper validation rules (`required|numeric|min:1024|max:65535`).

**Minor note**: With PORT max at 65535, PORT+2000 could exceed 65535 (e.g., PORT=64000 means server port=66000). Ideally max should be 63535. This is a low-priority edge case.

---

## Criterion 6: No dependency on Panel Mounts or custom Docker images

**PASS**

The PRD explicitly states in Constraints C2 and C3 that Panel Mounts and custom Docker images are forbidden. Goal G2/G3 and AC-5.6 reinforce this with explicit verification criteria ("No reference to `/rvs` or `ghcr.io/danpowell88/ravenshield_dedicatedserver` anywhere in the egg JSON"). The `docker_images` field uses only publicly available ptero-eggs yolks.

---

## Criterion 7: The PRD provides sufficient technical detail for downstream architecture and development

**PASS**

The PRD provides:
- Complete variable specifications (20 variables with types, defaults, rules, field types, descriptions)
- Exact egg metadata fields and JSON structure
- Install script requirements with 7 acceptance criteria
- Startup script behavior with 8 acceptance criteria
- Error handling matrix with expected behaviors
- Reference data (18 maps, 6 game types, 7 presets, port formula, done string)
- Non-functional requirements with specific targets
- File deliverable list
- Section 9 explicitly tasks the Architect with extracting remaining details from upstream

This is more than sufficient for a Developer to implement without ambiguity.

---

## Summary Table

| # | Criterion | Verdict |
|---|-----------|---------|
| 1 | Technical approach feasible | PASS |
| 2 | Docker image valid and public | PASS |
| 3 | Install script approach sound | PASS |
| 4 | Startup command approach sound | PASS |
| 5 | Port allocation correct | PASS |
| 6 | No Panel Mounts or custom images | PASS |
| 7 | Sufficient technical detail | PASS |

---

## Actionable Recommendations (non-blocking)

1. **Xvfb redundancy (REQ-3, item 3)**: The yolk entrypoint already starts Xvfb when `XVFB=1` (default). The `start.sh` script must NOT start its own Xvfb instance. Developer should skip REQ-3 item 3 and rely on `DISPLAY=:0` being pre-set by the yolk. This simplifies the startup script and avoids conflicts from dual Xvfb instances.

2. **PORT max value**: Consider changing PORT max from 65535 to 63535 to ensure PORT+2000 never exceeds valid port range. Low priority -- can be addressed in development.

3. **docker_images format tension**: The PRD correctly specifies key=value=URI per memory lesson, but also suggests offering multiple image options with human-readable labels (following wine generic pattern). These are contradictory. Developer should use key=value=URI with only `wine_latest` per the memory lesson.

---

## Final Verdict

**DONE** -- All seven criteria PASS. No blocking defects found. The PRD v2.0 is technically sound, feasible, and provides sufficient detail for implementation. The Wine yolk includes all required runtime dependencies (Wine, Xvfb, xauth). The install-script-plus-startup-script pattern is proven across the repository.
