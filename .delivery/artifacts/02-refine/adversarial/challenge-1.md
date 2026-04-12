# Adversarial Review: Raven Shield Dedicated Server Egg PRD

**Reviewer:** Adversarial Reviewer
**Date:** 2026-04-11
**PRD Version:** 1.0
**Overall Confidence Rating:** 2 / 5

---

## Executive Summary

The PRD contains several fundamental factual errors that will cause the implementation to fail without correction. The most critical is the startup command design: the Docker image uses a shell script entrypoint (`/entrypoint.sh`) that is the actual process manager for the server, and the `wine UCC.exe` invocation happens inside that script from a specific working directory (`/rvs/gamefiles/system`). The PRD's assumption that Pterodactyl can call the server binary directly (as required by repo policy) is in direct conflict with how the chosen Docker image works. This is a structural conflict between the image's architecture and the repo's no-shell-script rule. This alone blocks delivery as-is.

---

## Findings

---

### CRITICAL-1: The Docker Image's ENTRYPOINT IS a Shell Script — Direct Binary Invocation Is Impossible With This Image

**Rating:** CRITICAL

**Evidence:**
- The Dockerfile sets `ENTRYPOINT ["/entrypoint.sh"]` — the image is hard-wired to run its own shell script as PID 1.
- The entrypoint script: (a) extracts game files, (b) runs `crudini` to write INI configs, (c) initializes Wine and Xvfb, (d) changes directory to `/rvs/gamefiles/system`, then (e) runs `wine UCC.exe server -ini=... -serverini=... -log`.
- The repo's PR checklist explicitly requires: "You verify that the start command applied does not use a shell script."
- If Pterodactyl's startup command is set to call `/rvs/gamefiles/system/UCC.exe` or `wine /rvs/gamefiles/system/UCC.exe` directly, it bypasses the entrypoint entirely. Wine has not been initialized, Xvfb is not running, INI files have not been written, and game files may not have been extracted — the server will not start.

**PRD Gap:**
FR-6, FR-7, FR-8, and FR-9 all assume the server binary can be called directly. FR-8 acknowledges the path must be confirmed, but treats the problem as only a path-finding question. It does not acknowledge that the image's architecture (ENTRYPOINT script that does essential setup) is architecturally incompatible with the repo's no-shell-script requirement.

**Implication:**
Either (a) a different Docker image must be used that exposes a directly-invocable binary, (b) the image must be forked/modified so Pterodactyl's startup command can call the process directly, or (c) a waiver from the no-shell-script rule must be sought. None of these options are acknowledged in the PRD. The PRD must explicitly address this conflict before implementation begins.

---

### CRITICAL-2: GAME_PRESET Does Not Wire Into the Startup Command — It Writes INI Files Via the Entrypoint Script

**Rating:** CRITICAL

**Evidence:**
- The entrypoint script handles `GAME_PRESET` by calling `crudini` to set values in `Server.ini` (e.g., `MaxPlayers`, `ServerName`, map list, gametype list). This happens during the pre-launch setup phase inside `/entrypoint.sh`.
- The startup command (`wine UCC.exe server -ini=... -serverini=...`) does not reference `GAME_PRESET` at all.
- If Pterodactyl calls `wine UCC.exe` directly (bypassing the entrypoint), `GAME_PRESET` has zero effect — the INI writes never happen.

**PRD Gap:**
FR-9 shows startup command templates where `GAME_PRESET` is conspicuously absent from the command line (correctly, since it's not a command-line argument). But the PRD never explains the mechanism by which `GAME_PRESET` actually takes effect. Section 3.4 (FR-11) says the variable "Controls overall game mode preset" but gives no wiring detail. US-3 promises "run both COOP and adversarial game modes by changing a single preset variable" — but this is only true if the entrypoint script runs, which conflicts with CRITICAL-1.

**Implication:**
The `GAME_PRESET` variable is effectively a no-op if the repo's direct-binary-invocation requirement is met. The PRD must either (a) document that `GAME_PRESET` works via entrypoint-mediated INI manipulation and acknowledge the contradiction, or (b) redesign how preset switching works for direct-invocation scenarios.

---

### CRITICAL-3: The Directory Structure Requirement Conflicts With an Existing Directory in the Repo

**Rating:** CRITICAL

**Evidence:**
- The PRD specifies the egg must live at `rainbow_six/raven_shield/` (FR-1).
- The repository already contains a directory named `rainbow_six_3_raven_shield/` (confirmed in the repo's root listing and README). The README does not currently link this directory under a "Rainbow Six" section, but the directory exists.
- If this pre-existing directory already contains files (README.md is present), the implementer may create a duplicate structure or collide with existing work.

**PRD Gap:**
The PRD makes no mention of this existing directory. It does not say whether the work should go into `rainbow_six_3_raven_shield/` (the existing name) or `rainbow_six/raven_shield/` (a new nested structure). This will cause confusion at implementation time and potentially at PR review.

---

### MAJOR-1: The Startup Command Working Directory Is Not `/rvs` — It Is `/rvs/gamefiles/system`

**Rating:** MAJOR

**Evidence:**
- The entrypoint script performs `cd $GAMEFILES_DIR/system` (where `GAMEFILES_DIR=/rvs/gamefiles`) before invoking `wine UCC.exe`.
- FR-7 states: "The startup command must reference `/rvs` as the game directory."
- The actual game files are at `/rvs/gamefiles/`, not `/rvs/`. The `System/` directory is at `/rvs/gamefiles/system/`.
- The PRD's suggested Linux-native path `/rvs/System/UCC` is wrong — the image does not place files at `/rvs/System/` directly; it extracts to `/rvs/gamefiles/`.

**PRD Gap:**
FR-7 and FR-9 will produce a wrong path. Any startup command built on these specs will reference a path that does not exist in the container.

---

### MAJOR-2: The Image Requires Game Files to Be Provided by the Operator — Install Script Cannot Be a No-Op

**Rating:** MAJOR

**Evidence:**
- The README states: "The container expects `/rvs` to contain the root of your game files."
- The `BYO_GAMEFILES` environment variable controls whether the entrypoint script skips downloading game files. "Download from internet archive at runtime" is listed as a Future Enhancement — meaning it is NOT currently implemented.
- Game files must be provided by the operator (Raven Shield is a commercially licensed game). The `/rvs` volume must be populated before the server can start.

**PRD Gap:**
FR-6 / Section 3.6 says the install script can be "a no-op body with an informational echo" if the image is self-contained. The image is NOT self-contained for game files. A no-op install script will result in a server that always fails to start because game files are absent.

The PRD must define how game files reach the `/rvs` directory. Options include: (a) operator pre-populates `/rvs` on the host before panel allocation, (b) the install script copies from a provided source, or (c) the install script documents the manual step. None of these are specified.

---

### MAJOR-3: The "Done" String for `config.startup.done` Cannot Be the OpenRVS Pattern Without OpenRVS

**Rating:** MAJOR

**Evidence:**
- The entrypoint script watches for `"*OpenRVS is up to date*"` as its readiness signal.
- This pattern requires `INSTALL_OPENRVS=true` (the default in the Dockerfile's ENV).
- If an operator sets `INSTALL_OPENRVS=false`, this pattern will never appear and Pterodactyl will never consider the server started.
- OQ-2 (open question) correctly flags this as unresolved, but the PRD does not acknowledge the dependency on `INSTALL_OPENRVS`.

**PRD Gap:**
The PRD should either (a) mandate `INSTALL_OPENRVS=true` as a fixed/hidden variable, or (b) provide an alternative `done` string that works regardless of OpenRVS installation state, or (c) acknowledge this as a known limitation.

---

### MAJOR-4: CD Key / Licensing Risk Is Understated for a Commercial Game

**Rating:** MAJOR

**Evidence:**
- Raven Shield is a commercial game published by Ubisoft (2003). It is not abandonware nor freely licensed.
- The Docker image README provides no guidance on CD key requirements. No open-source or freeware server binary appears to exist for this game — the dedicated server executable is bundled with the retail game.
- The repo's own PR checklist item #2 asks: "Does your egg use a custom docker image?" and "Have you tried to use a generic image?" — implying scrutiny of custom images.
- OQ-3 identifies this as "High" priority but the PRD defers it entirely to the implementer with no fallback position.

**PRD Gap:**
The PRD should state a policy position: if a CD key or proof of purchase is required to legally run the server, this egg may not be acceptable for the public game-eggs repo regardless of technical correctness. The PRD acknowledges the risk in FR-19 (documentation) but does not gate delivery on a definitive answer. An implementer could complete all technical work only to have the PR rejected on licensing grounds.

---

### MAJOR-5: The Port Offset Convention Is Image-Specific, Not Standard Raven Shield

**Rating:** MAJOR

**Evidence:**
- Standard Raven Shield dedicated server documentation (and the original Ubisoft server config) uses port 7777 for game traffic and does not specify fixed offsets for beacon/query ports.
- The 8777 (+1000) and 9777 (+2000) offsets are specific to THIS Docker image's `entrypoint.sh` implementation (`SERVER_BEACON_PORT=$((PORT + 1000))` and `BEACON_PORT=$((PORT + 2000))`).
- These offsets are NOT a Raven Shield protocol standard; they are implementation choices by the image author.

**PRD Gap:**
FR-15 (port table) presents these as if they are universal Raven Shield port standards. The README should clarify these are image-specific offsets, not game-standard ports. If an operator uses a different server setup (not this Docker image), the port documentation would mislead them.

---

### MINOR-1: Map Default `Training_1` Is Likely Invalid for Multiplayer

**Rating:** MINOR

**Evidence:**
- `Training_1` is a single-player training mission map in Raven Shield. It was not designed for multiplayer use and may fail to load or behave incorrectly in a dedicated server context.
- Standard multiplayer maps use the `MP_` prefix (e.g., `MP_Village`, `MP_Airport`, `MP_Bank`).

**PRD Gap:**
FR-11 sets `MAP_1` default to `Training_1`. This is a bad default that will likely cause server errors on first boot if the map rotation cycles to slot 1. A valid multiplayer map (e.g., `MP_Airport`) should be used.

---

### MINOR-2: `GAMETYPE_0` Default Is COOP, But `GAME_PRESET` Default Is Also COOP — The Variables Are Redundant Without Explanation

**Rating:** MINOR

**Evidence:**
- `GAME_PRESET=COOP` and `GAMETYPE_0=R6Game.R6TeamAIGame` both set the server to COOP mode.
- If an operator changes `GAME_PRESET=ADVERSARIAL` but leaves `GAMETYPE_0=R6Game.R6TeamAIGame`, which takes precedence?
- In the entrypoint script, `GAME_PRESET` writes defaults to the INI file, but then individual `GAMETYPE_*` environment variables also write per-slot overrides. The interaction is not documented.

**PRD Gap:**
The PRD does not document the precedence or interaction between `GAME_PRESET` and `GAMETYPE_*` variables. An operator trying to run a mixed-mode rotation (COOP on map 1, adversarial on map 2) will have no guidance.

---

### MINOR-3: `M02_Span` Is a Campaign Mission Map, Not a Standard Multiplayer Map

**Rating:** MINOR

**Evidence:**
- `M02_Span` is the internal name for a single-player campaign mission in Raven Shield. It is not an `MP_` prefixed multiplayer map.
- Setting it as `MAP_2` default may cause the server to fail when loading that slot or produce unexpected gameplay.

**PRD Gap:**
Same as MINOR-1. Map defaults in FR-11 have not been validated against the actual Raven Shield multiplayer map list.

---

### MINOR-4: `features` Field: PRD Specifies `null` But Repo Examples Use `[]`

**Rating:** MINOR

**Evidence:**
- The PTDL_v2 schema appears in practice with `"features": []` (empty array) in other repo eggs (e.g., Americas Army: Proving Grounds).
- The PRD specifies `"features": null`.
- While both may be panel-compatible, there is an inconsistency with observed repo standards.

**PRD Gap:**
FR-4 should cross-reference the actual field value used in comparable repo eggs to avoid an unnecessary diff at PR review.

---

### MINOR-5: `config.startup.done` Pattern Depends on Wine Startup Output, Which Is Non-Deterministic

**Rating:** MINOR

**Evidence:**
- Wine initialization produces console output that varies by Wine version, display configuration, and whether Xvfb initializes cleanly.
- If the PRD's `done` string is set to something emitted before Wine is fully ready, Pterodactyl will mark the server as running before it is actually accepting connections.
- OQ-2 correctly defers this to the implementer, but no test methodology is specified.

**PRD Gap:**
The PRD should require the `done` string to be validated by observing actual container startup output (not assumed from documentation), and should specify that the test must confirm players can connect after the string appears.

---

## Summary Table

| ID | Severity | Topic |
|---|---|---|
| CRITICAL-1 | CRITICAL | Docker image ENTRYPOINT is a shell script; direct binary invocation is impossible with this image |
| CRITICAL-2 | CRITICAL | GAME_PRESET has no effect on startup command; its mechanism only works inside the entrypoint script |
| CRITICAL-3 | CRITICAL | Conflicting directory: `rainbow_six_3_raven_shield/` already exists in repo; PRD specifies `rainbow_six/raven_shield/` |
| MAJOR-1 | MAJOR | Startup command path is wrong; game files are at `/rvs/gamefiles/system/`, not `/rvs/System/` |
| MAJOR-2 | MAJOR | Install script cannot be a no-op; game files must be operator-provided and no mechanism is defined |
| MAJOR-3 | MAJOR | `config.startup.done` pattern depends on OpenRVS being installed; unacknowledged dependency |
| MAJOR-4 | MAJOR | Commercial licensing risk not gated; PR could be rejected after full implementation |
| MAJOR-5 | MAJOR | Port offsets are image-specific, not standard Raven Shield protocol |
| MINOR-1 | MINOR | `Training_1` is a SP map, likely invalid as a multiplayer MAP_1 default |
| MINOR-2 | MINOR | GAME_PRESET and GAMETYPE_* interaction/precedence undocumented |
| MINOR-3 | MINOR | `M02_Span` is a campaign map, not a valid multiplayer MAP_2 default |
| MINOR-4 | MINOR | `features: null` inconsistent with repo practice of `features: []` |
| MINOR-5 | MINOR | `done` string validation methodology not specified |

---

## Overall Confidence Rating: 2 / 5

The PRD is well-structured and thorough in its intent, but is built on a fundamental misunderstanding of the Docker image's architecture. The image is designed to run as its own process manager via a shell script entrypoint, which is directly incompatible with the repo's no-shell-script requirement. Until this structural conflict is resolved (either by changing the image or getting a policy waiver), no amount of refinement to the other requirements will unblock delivery. The three CRITICAL findings must be resolved before the PRD can be approved for implementation.
