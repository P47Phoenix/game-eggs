# Tom Clancy's Rainbow Six 3: Raven Shield

Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter released in 2003, featuring squad-based counter-terrorism missions with a strong emphasis on planning and precision. The dedicated server supports cooperative and adversarial multiplayer modes including Terrorist Hunt, Team Deathmatch, Bomb, Hostage Rescue, and Escort Pilot.

## Server Overview

This egg uses the Docker image [`ghcr.io/danpowell88/ravenshield_dedicatedserver`](https://ghcr.io/danpowell88/ravenshield_dedicatedserver), which is an externally maintained community image and is not managed by this repository. Its continued availability is not guaranteed.

On first startup the container automatically downloads the game files (~500 MB) from [archive.org](https://archive.org) via the [OpenRVS](https://github.com/OpenRVS/openrvs-registry) community patch. No CD key or Steam login is required.

> [!IMPORTANT]
>
> The entrypoint script (`/entrypoint.sh`) is baked into the Docker image. It is **not** a user-managed script and cannot be replaced or edited from the panel. All server configuration is done exclusively through environment variables.

> [!NOTE]
>
> The first startup may take several minutes while the game files are being downloaded from archive.org (approximately 500 MB). Subsequent starts are fast.

## Installation / Setup

1. In your Pterodactyl panel, navigate to **Admin > Nests** and select or create a nest for the game.
2. Click **Import Egg** and upload `egg-rainbow-six-3-raven-shield.json` from this directory.
3. Create a new server using the imported egg and allocate **three UDP ports** (see the port table below).
4. Configure environment variables as needed (see the reference table below).
5. Start the server. The first start will download game files automatically — allow several minutes before the server becomes available.

## Server Ports

Three UDP ports must be allocated in the panel. The beacon ports are derived from the primary game port.

| Port | Default | Protocol | Purpose |
|------|---------|----------|---------|
| PORT | 7777 | UDP | Primary game traffic |
| PORT + 1000 | 8777 | UDP | Server beacon (OpenRVS / GameSpy) |
| PORT + 2000 | 9777 | UDP | Beacon port |

> [!WARNING]
>
> If you change `PORT` from the default `7777`, you must re-allocate all three derived ports in the panel accordingly (e.g., if `PORT=7800`, allocate 7800, 8800, and 9800).

## Environment Variables

| Variable | Default | Allowed Values | Description |
|----------|---------|----------------|-------------|
| `GAME_PRESET` | `COOP` | `COOP`, `ADVERSARIAL`, `DEATHMATCH`, `TEAMDEATHMATCH`, `BOMB`, `HOSTAGERESCUE`, `ESCORTPILOT` | Server game mode preset. Sets map rotation, game types, and server settings. |
| `PORT` | `7777` | 1024–65535 | Primary UDP game port. |
| `NAME` | `Pterodactyl Raven Shield` | Any string (max 64 chars) | Server name displayed in the server browser. |
| `MAX_PLAYERS` | `16` | 1–64 | Maximum number of players allowed on the server. |
| `MAP_0` | `Airport` | See map list below | Map name for rotation slot 0. |
| `GAMETYPE_0` | `R6Game.R6TerroristHuntCoopGame` | See game type list below | Game type for map rotation slot 0. |
| `MAP_1` | `Alpines` | See map list below | Map name for rotation slot 1. Leave blank to disable. |
| `GAMETYPE_1` | `R6Game.R6TerroristHuntCoopGame` | See game type list below | Game type for map rotation slot 1. Leave blank to disable. |
| `MAP_2` | `Shipyard` | See map list below | Map name for rotation slot 2. Leave blank to disable. |
| `GAMETYPE_2` | `R6Game.R6TerroristHuntCoopGame` | See game type list below | Game type for map rotation slot 2. Leave blank to disable. |
| `MAP_3` | _(blank)_ | See map list below | Map name for rotation slot 3. Leave blank to disable. |
| `GAMETYPE_3` | _(blank)_ | See game type list below | Game type for map rotation slot 3. Leave blank to disable. |
| `MAP_4` | _(blank)_ | See map list below | Map name for rotation slot 4. Leave blank to disable. |
| `GAMETYPE_4` | _(blank)_ | See game type list below | Game type for map rotation slot 4. Leave blank to disable. |
| `MAP_5` | _(blank)_ | See map list below | Map name for rotation slot 5. Leave blank to disable. |
| `GAMETYPE_5` | _(blank)_ | See game type list below | Game type for map rotation slot 5. Leave blank to disable. |
| `ADMIN_PASSWORD` | _(blank)_ | Any string (max 32 chars) | Password for remote admin access. Leave blank to disable. |
| `GAME_PASSWORD` | _(blank)_ | Any string (max 32 chars) | Password players must enter to join. Leave blank for a public server. |
| `INTERNET_SERVER` | `true` | `true`, `false` | Set to `true` to register the server publicly on the server browser. Set to `false` for a LAN-only server. |
| `INSTALL_OPENRVS` | `true` | `true`, `false` | Set to `true` to install the OpenRVS community patch, which restores server browser functionality and fixes several bugs. Recommended. |

## Game Presets

The `GAME_PRESET` variable selects a pre-configured server profile. Each preset configures appropriate map rotations, game types, and server settings for that mode.

| Preset | Description |
|--------|-------------|
| `COOP` | Cooperative Terrorist Hunt mode. Players work together against AI enemies. Default preset. |
| `ADVERSARIAL` | General adversarial (PvP) preset. |
| `DEATHMATCH` | Free-for-all deathmatch. |
| `TEAMDEATHMATCH` | Team-based deathmatch. |
| `BOMB` | Teams compete to plant or defuse a bomb. |
| `HOSTAGERESCUE` | Attackers rescue hostages; defenders protect them. |
| `ESCORTPILOT` | Attackers escort a pilot to an extraction point; defenders stop them. |

## Valid Map Names

Map names are **case-sensitive**. Using an incorrect map name will cause the server to fail to load that map or may prevent startup. The valid map names are:

| Map Name |
|----------|
| `Airport` |
| `Alpines` |
| `Bank` |
| `Garage` |
| `Import_Export` |
| `Island_Dawn` |
| `MeatPacking` |
| `Mountain_High` |
| `Oil_Refinery` |
| `Parade` |
| `Peaks` |
| `Penthouse` |
| `Presidio` |
| `Prison` |
| `Shipyard` |
| `Streets` |
| `Training` |
| `Warehouse` |

## Valid Game Types

Game type values are **case-sensitive** and must be entered exactly as shown. Using an incorrect value will cause a startup failure.

| Game Type Value | Plain-English Name |
|----------------|--------------------|
| `R6Game.R6TerroristHuntCoopGame` | Terrorist Hunt (Cooperative) |
| `R6Game.R6TeamBomb` | Team Bomb |
| `R6Game.R6HostageRescueAdvGame` | Hostage Rescue (Adversarial) |
| `R6Game.R6TeamDeathMatchGame` | Team Deathmatch |
| `R6Game.R6EscortPilotGame` | Escort Pilot |
| `R6Game.R6DeathMatch` | Deathmatch (Free-for-all) |

## Known Issues / Operator Requirements

### /rvs Volume Mount Requirement

**Symptoms:** The container crashes immediately on startup with exit code 1 and produces no console output (or a brief "permission denied" message before exiting).

**Root cause:** The `GAMEFILES_DIR` path is hardcoded to `/rvs/gamefiles` inside the Docker image (`ghcr.io/danpowell88/ravenshield_dedicatedserver`). Pterodactyl Wings mounts server data at `/home/container` by default. Because the container process expects to write game files to `/rvs` but Wings places the data volume at `/home/container`, the entrypoint fails immediately with a permission denied error on `/rvs`.

**Fix:** Use the Pterodactyl Panel Mounts feature to map a host path into the container at `/rvs`.

1. In the Panel admin, go to **Admin → Mounts** and create a new Mount:
   - **Source:** the host path you want to use for game data (e.g. `/var/lib/pterodactyl/volumes/<server-uuid>` or a dedicated path)
   - **Target:** `/rvs`
   - **Read Only:** No
2. On the Mount's detail page, assign the Mount to this **Egg** and the **Node** running this server.
3. Open the server in the admin panel, go to **Mounts**, and click **+** to assign the mount to the server instance.
4. In Wings `config.yml` on that node, ensure the source path is listed under `allowed_mounts`. Restart Wings after editing.
5. Start the server — it will now write game files to `/rvs` successfully.

> [!NOTE]
>
> This is an image-level limitation imposed by the third-party Docker image, not a defect in this Pterodactyl egg. The egg cannot override the hardcoded `/rvs` path without modifying the image itself.

## Known Limitations

- **External Docker image.** The image `ghcr.io/danpowell88/ravenshield_dedicatedserver` is maintained by a third party and is not part of this repository. Its availability, updates, and continued compatibility are not guaranteed.
- **Map and game type values are not validated by the panel.** The `MAP_*` and `GAMETYPE_*` fields accept any string; an incorrect value will cause a startup failure rather than a panel validation error. Double-check spelling and case before starting the server.
- **Changing PORT requires re-allocating all three ports.** The server uses `PORT`, `PORT+1000`, and `PORT+2000`. All three must be allocated in the panel. Changing `PORT` without updating the other two allocations will prevent the server from starting correctly.
- **Entrypoint script is not user-editable.** The `/entrypoint.sh` startup script is baked into the Docker image. It cannot be replaced or modified from the panel. All configuration must be done through environment variables.
