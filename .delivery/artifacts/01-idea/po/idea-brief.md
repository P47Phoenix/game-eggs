## Idea Brief

**Project Type**: GREENFIELD
**Date**: 2026-04-12
**Pipeline**: run-2026-04-12-r6egg

### Problem Statement
The current Rainbow Six 3: Raven Shield Pterodactyl egg uses the pre-built Docker image `ghcr.io/danpowell88/ravenshield_dedicatedserver` which hardcodes game data to `/rvs`. Pterodactyl mounts server data at `/home/container` and runs containers as a non-root UID, causing `Permission denied` crashes on every start (`mkdir: cannot create directory '/rvs/setup': Permission denied`). Panel Mount workarounds proved unreliable across different Wings environments (especially containerized Wings on TrueNAS SCALE).

The PO directive: stop fighting the upstream Docker image. Build a proper egg from scratch that works like every other egg in the repo — use a standard base image with Wine runtime, write egg install/startup scripts that put everything in `/home/container`.

### Target Users
- **Pterodactyl panel operators**: Server hosts who want to run Rainbow Six 3: Raven Shield dedicated servers through the Pterodactyl panel without custom Docker images, Panel Mounts, or Wings config changes.

### Goals
1. Create a proper PTDL_v2 egg that starts successfully on any standard Pterodactyl + Wings installation
2. Use a standard base image (same as or equivalent to the upstream Dockerfile's base — Ubuntu + Wine + Xvfb) — no custom Docker image builds required
3. Egg install script handles game file download and setup into `/home/container`
4. Egg startup command launches the server from `/home/container` via Wine
5. Retain all user-configurable variables: MAP_0-5, GAMETYPE_0-5, GAME_PRESET, PORT, NAME, MAX_PLAYERS, passwords, INTERNET_SERVER, INSTALL_OPENRVS

### Constraints
- Must use PTDL_v2 egg JSON format
- Must NOT require Panel Mounts or Wings `allowed_mounts` configuration
- Must NOT require a custom-built Docker image (use existing public base images)
- Game data must live in `/home/container` (Pterodactyl's default mount point)
- Must work on containerized Wings (e.g., TrueNAS SCALE) — not just direct-install Wings
- The upstream repo `github.com/danpowell88/ravenshield_dedicatedserver` is the reference for understanding install steps, server launch, env vars, and OpenRVS patching

### Initial Scope
- Analyze the upstream Dockerfile and entrypoint.sh to extract: base image, game file download process, Wine/Xvfb setup, server config generation, server launch command, OpenRVS patching
- Select or identify a suitable public base Docker image with Wine + Xvfb pre-installed
- Write an egg install script (`scripts.installation.script`) that replicates the game setup in `/home/container`
- Write a startup command that launches the server via Wine from `/home/container`
- Create the full egg JSON with all variables, proper validation rules, and descriptions
- Create a README with setup instructions
- Test on the user's TrueNAS SCALE Pterodactyl instance

### Out of Scope (initial)
- Modifying or forking the upstream Docker image
- Contributing fixes upstream to `danpowell88/ravenshield_dedicatedserver`
- Supporting other game variants or mods beyond base Raven Shield + OpenRVS
- Panel Mount documentation (no longer needed)

### Reference Material
- Upstream Docker repo: `github.com/danpowell88/ravenshield_dedicatedserver`
- Previous pipeline artifacts from `run-2026-04-12-r6fix` (BUG_FIX) contain confirmed details about:
  - Valid map names (18 maps): Airport, Alpines, Bank, Garage, Import_Export, Island_Dawn, MeatPacking, Mountain_High, Oil_Refinery, Parade, Peaks, Penthouse, Presidio, Prison, Shipyard, Streets, Training, Warehouse
  - Valid game types (6): R6Game.R6TerroristHuntCoopGame, R6Game.R6TeamBomb, R6Game.R6HostageRescueAdvGame, R6Game.R6TeamDeathMatchGame, R6Game.R6EscortPilotGame, R6Game.R6DeathMatch
  - Port formula: PORT (game), PORT+1000 (beacon), PORT+2000 (server)
  - Done string: "OpenRVS is up to date"
  - entrypoint.sh supports MAP_0–MAP_31 and GAMETYPE_0–GAMETYPE_31 via loop
  - Game files downloaded from archive.org; OpenRVS from GitHub releases
