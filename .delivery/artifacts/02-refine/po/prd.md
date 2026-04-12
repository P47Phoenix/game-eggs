# Product Requirements Document: Rainbow Six 3: Raven Shield Pterodactyl Egg (Rebuilt)

**Version**: 2.0
**Date**: 2026-04-12
**Author**: Product Owner
**Pipeline**: run-2026-04-12-r6egg
**Project Type**: GREENFIELD
**Supersedes**: PRD v1.1 (pre-built Docker image approach -- abandoned due to /rvs permission crashes)

---

## 1. Problem Statement

The current Rainbow Six 3: Raven Shield egg uses the pre-built Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver` which hardcodes game data to `/rvs`. Pterodactyl mounts server data at `/home/container` and runs containers as a non-root UID, causing immediate `Permission denied` crashes on every start (`mkdir: cannot create directory '/rvs/setup': Permission denied`). Panel Mount workarounds proved unreliable across different Wings environments (especially containerized Wings on TrueNAS SCALE).

**Solution**: Build a proper PTDL_v2 egg from scratch that uses a standard Wine base image from the ptero-eggs yolks, installs game files into `/home/container` via the egg install script, and launches the server from `/home/container` via a startup command. This is how every other egg in the repo works.

---

## 2. Goals

| # | Goal | Success Metric |
|---|------|---------------|
| G1 | Server starts on any standard Pterodactyl + Wings installation | Server reaches "OpenRVS is up to date" done string without errors |
| G2 | No custom Docker images required | `docker_images` field uses only publicly available ptero-eggs yolks images |
| G3 | No Panel Mounts or Wings config changes required | Zero references to `/rvs` or `allowed_mounts`; all data in `/home/container` |
| G4 | All game data lives in `/home/container` | All file operations target `/home/container` (runtime) or `/mnt/server` (install-time) |
| G5 | Works on containerized Wings (TrueNAS SCALE) | Successful start on user's TrueNAS SCALE instance |

---

## 3. Constraints

| # | Constraint |
|---|-----------|
| C1 | Must use PTDL_v2 egg JSON format |
| C2 | Must NOT require Panel Mounts or Wings `allowed_mounts` configuration |
| C3 | Must NOT require a custom-built Docker image -- use existing ptero-eggs yolks |
| C4 | Game data must live in `/home/container` (Pterodactyl's default mount point) |
| C5 | Must work on containerized Wings (e.g., TrueNAS SCALE) |
| C6 | The upstream repo `github.com/danpowell88/ravenshield_dedicatedserver` is the authoritative reference for install steps, server launch, env vars, and OpenRVS patching |
| C7 | No user-managed wrapper scripts outside `/home/container` -- startup logic must be either inline in the startup field or a script deployed by the install script into `/home/container` |

---

## 4. Requirements

### REQ-1: Docker Image Selection

**Description**: The egg must use a standard publicly-available Docker image that includes Wine. The game requires Xvfb (virtual framebuffer) at runtime.

**Specification**:
- Primary image: `ghcr.io/ptero-eggs/yolks:wine_latest`
- The `docker_images` key must equal the value (both are the image URI)
- The Architect agent MUST verify that the selected yolk includes Xvfb or that Xvfb can be started without root privileges
- If wine_latest does NOT include Xvfb, the install script must install xvfb into `/home/container` or the startup must use an alternative approach
- Offer multiple image options if appropriate (Latest, Staging, Devel) following the pattern used by `wine/generic/egg-wine-generic.json`

**docker_images format** (key = value = image URI):
```json
"docker_images": {
    "ghcr.io/ptero-eggs/yolks:wine_latest": "ghcr.io/ptero-eggs/yolks:wine_latest"
}
```

**Acceptance Criteria**:
- AC-1.1: `docker_images` value is a publicly available image URI (no custom builds)
- AC-1.2: The image includes Wine capable of running Windows executables
- AC-1.3: Xvfb is available at runtime (either pre-installed in the yolk or made available by install/startup logic)
- AC-1.4: The container runs as non-root (UID 1000) without permission errors

---

### REQ-2: Install Script

**Description**: The egg install script (`scripts.installation.script`) must download and set up all game files into `/mnt/server` (which maps to `/home/container` at runtime).

**Specification**:
The install script must:
1. Install required tools: `curl`, `wget`, `unzip`, `tar` (and any other dependencies needed for download/extraction)
2. Create `/mnt/server` if it does not exist
3. Download game files from archive.org (the Architect agent must extract the exact URL from the upstream Dockerfile/entrypoint.sh)
4. Extract game files to `/mnt/server`
5. If `INSTALL_OPENRVS` is `true`: download the latest OpenRVS release from GitHub (`github.com/OpenRVS-devs/OpenRVS/releases`) and patch the game files in place
6. Deploy a startup script (`/mnt/server/start.sh`) that handles server configuration generation and launch at runtime
7. Set appropriate file permissions (no root-owned files that would block the runtime user)
8. The script should be idempotent -- safe to re-run without corrupting an existing installation (skip download if files already exist)

**Install container**: `ghcr.io/ptero-eggs/installers:debian` with entrypoint `bash`

**Acceptance Criteria**:
- AC-2.1: Running the install script on a fresh server populates `/mnt/server` with all required game files (UCC.exe, system directory, maps, etc.)
- AC-2.2: Game files are accessible by the runtime container user (no permission denied errors)
- AC-2.3: OpenRVS patch files are correctly placed when `INSTALL_OPENRVS=true`
- AC-2.4: Re-running the install script does not break an existing installation
- AC-2.5: Install script exits with code 0 on success, non-zero on failure with meaningful error
- AC-2.6: Install script produces meaningful log output showing progress (download status, extraction, patching steps)
- AC-2.7: A startup helper script (`start.sh`) is deployed to `/mnt/server`

---

### REQ-3: Startup Command

**Description**: The egg startup command must launch the Raven Shield dedicated server from `/home/container` using Wine with a virtual framebuffer.

**Specification**:
The startup field value: `bash /home/container/start.sh`
(The Architect may refine this if a better approach is found, but the script must live within `/home/container`.)

The startup script must:
1. Validate GAME_PRESET -- exit with error listing valid values if invalid
2. Check that game files exist -- exit with "reinstall" message if UCC.exe (or equivalent) is missing
3. Start Xvfb on a virtual display (e.g., `:99`) or use `xvfb-run`
4. Set `DISPLAY` environment variable for Wine
5. Generate server configuration files (RavenShield.ini or equivalent) from environment variables:
   - Server name from `NAME`
   - Max players from `MAX_PLAYERS`
   - Port configuration from `PORT` (game), `PORT+1000` (beacon), `PORT+2000` (server)
   - Internet server flag from `INTERNET_SERVER`
   - Admin/game passwords from `ADMIN_PASSWORD`/`GAME_PASSWORD`
   - Map rotation from `MAP_0` through `MAP_5` (skip empty/unset slots)
   - Game type rotation from `GAMETYPE_0` through `GAMETYPE_5` (skip empty/unset slots)
6. Launch the server executable via Wine with appropriate parameters
7. The Wine process must stay in the foreground so Pterodactyl can manage its lifecycle via ^C

**Acceptance Criteria**:
- AC-3.1: Server process starts without permission errors
- AC-3.2: Server outputs "OpenRVS is up to date" (the done string) when successfully started with INSTALL_OPENRVS=true
- AC-3.3: Server listens on the configured PORT (UDP)
- AC-3.4: Server beacon responds on PORT+1000
- AC-3.5: Server responds on PORT+2000
- AC-3.6: The Wine process runs in the foreground (Pterodactyl can send ^C to stop it)
- AC-3.7: Invalid GAME_PRESET value causes exit with descriptive error message listing valid values
- AC-3.8: Missing game files (install not run) causes exit with message directing user to reinstall

---

### REQ-4: Egg Variables

**Description**: All configurable server parameters must be exposed as egg variables with proper defaults, validation rules, and descriptions.

#### Variable Specifications (20 total):

| # | Name | env_variable | Default | Rules | Viewable | Editable | Field Type |
|---|------|-------------|---------|-------|----------|----------|------------|
| 1 | Game Preset | GAME_PRESET | COOP | required\|string\|in:COOP,ADVERSARIAL,DEATHMATCH,TEAMDEATHMATCH,BOMB,HOSTAGERESCUE,ESCORTPILOT | true | true | text |
| 2 | Server Port | PORT | 7777 | required\|numeric\|min:1024\|max:65535 | true | true | text |
| 3 | Server Name | NAME | Pterodactyl Raven Shield | required\|string\|max:64 | true | true | text |
| 4 | Max Players | MAX_PLAYERS | 16 | required\|numeric\|min:1\|max:64 | true | true | text |
| 5 | Map Slot 0 | MAP_0 | Airport | required\|string\|max:64 | true | true | text |
| 6 | Game Type Slot 0 | GAMETYPE_0 | R6Game.R6TerroristHuntCoopGame | required\|string\|max:128 | true | true | text |
| 7 | Map Slot 1 | MAP_1 | Alpines | nullable\|string\|max:64 | true | true | text |
| 8 | Game Type Slot 1 | GAMETYPE_1 | R6Game.R6TerroristHuntCoopGame | nullable\|string\|max:128 | true | true | text |
| 9 | Map Slot 2 | MAP_2 | Shipyard | nullable\|string\|max:64 | true | true | text |
| 10 | Game Type Slot 2 | GAMETYPE_2 | R6Game.R6TerroristHuntCoopGame | nullable\|string\|max:128 | true | true | text |
| 11 | Map Slot 3 | MAP_3 | (empty) | nullable\|string\|max:64 | true | true | text |
| 12 | Game Type Slot 3 | GAMETYPE_3 | (empty) | nullable\|string\|max:128 | true | true | text |
| 13 | Map Slot 4 | MAP_4 | (empty) | nullable\|string\|max:64 | true | true | text |
| 14 | Game Type Slot 4 | GAMETYPE_4 | (empty) | nullable\|string\|max:128 | true | true | text |
| 15 | Map Slot 5 | MAP_5 | (empty) | nullable\|string\|max:64 | true | true | text |
| 16 | Game Type Slot 5 | GAMETYPE_5 | (empty) | nullable\|string\|max:128 | true | true | text |
| 17 | Admin Password | ADMIN_PASSWORD | (empty) | nullable\|string\|max:64 | true | true | text |
| 18 | Game Password | GAME_PASSWORD | (empty) | nullable\|string\|max:64 | true | true | text |
| 19 | Internet Server | INTERNET_SERVER | true | required\|string\|in:true,false | true | true | text |
| 20 | Install OpenRVS | INSTALL_OPENRVS | true | required\|string\|in:true,false | true | true | text |

#### Variable Description Requirements:

- **GAME_PRESET**: Must list all 7 valid preset values. Description: "Server game mode preset. Valid values: COOP, ADVERSARIAL, DEATHMATCH, TEAMDEATHMATCH, BOMB, HOSTAGERESCUE, ESCORTPILOT"
- **PORT**: Must explain three-port formula. Description: "Primary UDP game port. The server also requires PORT+1000 (beacon, default 8777) and PORT+2000 (server, default 9777). All three ports must be allocated in the panel."
- **MAP_* descriptions**: Must list all 18 valid map names: Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse
- **GAMETYPE_* descriptions**: Must list all 6 valid game type class names with human-readable labels. Must include warning: "An invalid game type will cause the server to exit on startup with an error."
- **MAP_0/GAMETYPE_0**: Use `required` in rules
- **MAP_1-5/GAMETYPE_1-5**: Use `nullable` in rules

**Acceptance Criteria**:
- AC-4.1: All 20 variables are present in the egg JSON `variables` array
- AC-4.2: MAP_0 and GAMETYPE_0 use `required` in rules; slots 1-5 use `nullable`
- AC-4.3: All GAMETYPE_* descriptions include the invalid-value startup-exit warning
- AC-4.4: PORT description documents the three-port formula
- AC-4.5: Default values match the table above exactly
- AC-4.6: All validation rules match the table above exactly
- AC-4.7: All `field_type` values are `"text"`
- AC-4.8: No variable has `user_editable: true` without also having `user_viewable: true`

---

### REQ-5: Egg Metadata and Configuration

**Description**: The egg JSON must contain proper metadata, config, and description fields.

**Specification**:
- `_comment`: `"DO NOT EDIT: FILE GENERATED AUTOMATICALLY BY PTERODACTYL PANEL - PTERODACTYL.IO"`
- `meta.version`: `PTDL_v2`
- `meta.update_url`: `null`
- `exported_at`: ISO 8601 datetime string
- `name`: `Rainbow Six 3: Raven Shield`
- `author`: `michaelconne@gmail.com`
- `features`: `null`
- `file_denylist`: `[]`
- `startup`: `bash /home/container/start.sh` (or Architect-refined equivalent)
- `config.startup`: `{"done": "OpenRVS is up to date"}`
- `config.stop`: `^C`
- `config.logs`: `{}`
- `config.files`: `{}`

**Description field** (operator warnings MUST be the VERY FIRST content):
```
WARNING: This egg requires three UDP ports allocated in the panel: PORT (game, default 7777), PORT+1000 (beacon, default 8777), PORT+2000 (server, default 9777). | Tom Clancy's Rainbow Six 3: Raven Shield dedicated server running via Wine on a standard Pterodactyl yolk. Supports cooperative and adversarial multiplayer modes including Terrorist Hunt, Team Deathmatch, Bomb, Hostage Rescue, and Escort Pilot. Uses the OpenRVS community patch for server browser compatibility.
```

**Acceptance Criteria**:
- AC-5.1: Description field begins with "WARNING:" operator warning (not game description)
- AC-5.2: Done string is exactly `OpenRVS is up to date`
- AC-5.3: Stop command is `^C`
- AC-5.4: `docker_images` key equals value (image URI on both sides)
- AC-5.5: `meta.version` is `PTDL_v2`
- AC-5.6: No reference to `/rvs` or `ghcr.io/danpowell88/ravenshield_dedicatedserver` anywhere in the egg JSON

---

### REQ-6: Error Handling

**Description**: The startup must handle error cases gracefully with clear messages to stdout (visible in Pterodactyl console).

**Specification**:

| Error Case | Expected Behavior |
|-----------|-------------------|
| Invalid GAME_PRESET (not in allowed list) | Exit with error listing valid presets |
| Missing game files (install not run) | Exit with error: "Game files not found. Please run the installer (Reinstall Server)." |
| Invalid GAMETYPE_* value | Server exits on startup with error (documented in variable description as warning) |
| Invalid MAP_* value | Server may fail to load map (game engine handles; documented in README) |
| PORT below 1024 or above 65535 | Panel validation prevents this (rules field) |
| Xvfb fails to start | Exit with error indicating virtual display failure |

**Acceptance Criteria**:
- AC-6.1: Invalid GAME_PRESET causes exit with descriptive error listing valid values
- AC-6.2: Missing game executable causes exit with "reinstall" guidance message
- AC-6.3: All error messages are written to stdout (visible in Pterodactyl console)
- AC-6.4: Error exits use non-zero exit code

---

## 5. Non-Functional Requirements

| # | Requirement | Target |
|---|-------------|--------|
| NFR-1 | Install time | Under 10 minutes on a 100 Mbps connection (game files ~1-2 GB from archive.org) |
| NFR-2 | Startup time | Server reaches done string within 60 seconds of container start |
| NFR-3 | Memory usage | Under 512 MB RAM at steady state |
| NFR-4 | Disk usage | Under 3 GB for full installation including OpenRVS |
| NFR-5 | Idempotent install | Re-running install does not corrupt or duplicate data |
| NFR-6 | JSON validity | File passes `jq .` or `python -m json.tool` without errors |
| NFR-7 | Encoding | UTF-8, Unix line endings |

---

## 6. Success Metrics

| Metric | Definition | Target |
|--------|-----------|--------|
| Clean start | Server reaches "OpenRVS is up to date" without errors | 100% of starts after successful install |
| No permission errors | Zero `Permission denied` in server output | 0 occurrences |
| Port listening | UDP port PORT responds to game client queries | Within 60s of start |
| Player connect | A game client can connect and join the server | Verified in UAT |
| Install success | Install script completes without error on fresh allocation | 100% success rate |
| TrueNAS SCALE | Works on containerized Wings without workarounds | Verified on user's instance |
| No Panel Mounts | Server runs without any Panel Mount configuration | Verified -- no /rvs references |

---

## 7. File Deliverables

| File | Purpose |
|------|---------|
| `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` | The PTDL_v2 egg definition (replaces current version) |
| `rainbow_six_3_raven_shield/README.md` | Documentation for the egg (replaces current version) |
| Root `README.md` update | New row in games table (alphabetical order) |

---

## 8. Out of Scope

- Modifying or forking the upstream Docker image (`danpowell88/ravenshield_dedicatedserver`)
- Supporting more than 6 map rotation slots (upstream supports 32; 6 is sufficient for MVP)
- Custom mod support beyond OpenRVS
- Automated game file update mechanism
- Contributing fixes upstream
- CI/CD pipeline changes to the game-eggs repository
- Multiple egg variants per game mode
- Panel Mount documentation (no longer applicable)

---

## 9. Technical Notes for Architect

The Architect agent MUST:

1. **Analyze the upstream source** at `github.com/danpowell88/ravenshield_dedicatedserver`:
   - Extract the exact archive.org download URL for game files from the Dockerfile or entrypoint.sh
   - Identify the file structure expected by the game (where UCC.exe lives, config file paths, directory layout)
   - Extract the Wine/Xvfb launch command from entrypoint.sh
   - Document the OpenRVS patching process (source URL, which files, where they go)
   - Extract the INI configuration generation logic (what fields, what format, what tool -- crudini or sed)
   - Identify the GAME_PRESET to map/gametype mapping logic

2. **Verify the Wine yolk**: Confirm `ghcr.io/ptero-eggs/yolks:wine_latest` includes Xvfb or determine how to provide it at runtime without root

3. **Decide startup approach**: The startup script (`start.sh`) is deployed by the install script into `/home/container`. This is the same pattern used by many eggs in the repo. The script handles config generation + server launch.

4. **Port handling**: Ensure the startup script correctly calculates PORT+1000 and PORT+2000 and passes them to the server configuration

5. **GAME_PRESET mapping**: Document what each preset means in terms of default map/gametype selections (from entrypoint.sh analysis). If GAME_PRESET overrides MAP_*/GAMETYPE_* or merely provides defaults, document which behavior applies.

6. **No references to old approach**: The new egg must have zero references to `/rvs`, `ghcr.io/danpowell88/ravenshield_dedicatedserver`, or Panel Mounts.

---

## 10. Valid Reference Data

### Maps (18 total):
Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse

### Game Types (6 total):
| Class Name | Human Label |
|-----------|-------------|
| R6Game.R6TerroristHuntCoopGame | Terrorist Hunt (Co-op) |
| R6Game.R6TeamBomb | Team Bomb |
| R6Game.R6HostageRescueAdvGame | Hostage Rescue |
| R6Game.R6TeamDeathMatchGame | Team Deathmatch |
| R6Game.R6EscortPilotGame | Escort the Pilot |
| R6Game.R6DeathMatch | Deathmatch |

### Port Formula:
- PORT (default 7777): Primary game UDP port
- PORT+1000 (default 8777): Beacon port
- PORT+2000 (default 9777): Server port

### Done String:
```
OpenRVS is up to date
```

### Game Presets:
COOP, ADVERSARIAL, DEATHMATCH, TEAMDEATHMATCH, BOMB, HOSTAGERESCUE, ESCORTPILOT
