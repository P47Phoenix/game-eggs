# Release Notes — Rainbow Six 3: Raven Shield Egg Update

**Date:** 2026-04-11
**File:** `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json`
**README:** `rainbow_six_3_raven_shield/README.md`

---

## Summary

This update expands the map/game type rotation from 3 slots to 6, standardizes variable descriptions and validation rules across all rotation slots, surfaces the /rvs volume mount requirement as a prominent operator warning in the egg description, and adds a Known Issues section to the README documenting the /rvs crash, its root cause, and the Wings bind-mount workaround.

---

## Changes

### 1. Map/Game Type Rotation Expanded from 3 Slots to 6

The egg previously exposed three map/game type pairs (`MAP_0`/`GAMETYPE_0` through `MAP_2`/`GAMETYPE_2`). This update adds three additional pairs:

| New Variable | Default | Notes |
|---|---|---|
| `MAP_3` | _(blank)_ | Optional; leave blank to stop rotation here |
| `GAMETYPE_3` | _(blank)_ | Required if `MAP_3` is set |
| `MAP_4` | _(blank)_ | Optional |
| `GAMETYPE_4` | _(blank)_ | Required if `MAP_4` is set |
| `MAP_5` | _(blank)_ | Optional |
| `GAMETYPE_5` | _(blank)_ | Required if `MAP_5` is set |

Slots 3–5 default to blank and are optional. Operators fill only the slots they need; leaving a slot blank stops the rotation at that point.

### 2. Full Valid-Value Lists Inlined in All Slot 1–5 Descriptions

PTDL_v2 does not support a dropdown field type. Previously, slots 1–5 described their values by cross-reference only. All `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` descriptions now include the complete inline list of valid values, matching the detail level of slot 0. Operators no longer need to consult a separate reference to enter correct values.

**Map slots (1–5)** include the full list of 18 case-sensitive map names:
`Airport`, `Alpines`, `Bank`, `Garage`, `Import_Export`, `Island_Dawn`, `MeatPacking`, `Mountain_High`, `Oil_Refinery`, `Parade`, `Peaks`, `Penthouse`, `Presidio`, `Prison`, `Shipyard`, `Streets`, `Training`, `Warehouse`

**Game type slots (1–5)** include all 6 valid class strings with their plain-English names:
- `R6Game.R6TerroristHuntCoopGame` — Terrorist Hunt (Co-op)
- `R6Game.R6TeamBomb` — Team Bomb
- `R6Game.R6HostageRescueAdvGame` — Hostage Rescue
- `R6Game.R6TeamDeathMatchGame` — Team Deathmatch
- `R6Game.R6EscortPilotGame` — Escort the Pilot
- `R6Game.R6DeathMatch` — Deathmatch

### 3. GAMETYPE_* Validation Rule Standardized to max:128

Previously, `GAMETYPE_1` and `GAMETYPE_2` used `max:64` in their validation rules. All `GAMETYPE_*` fields (slots 0–5) now use `max:128`. This aligns with the actual maximum length of valid game type class strings, which exceed 64 characters in some cases, and prevents the panel from incorrectly rejecting valid values.

| Variable | Old max | New max |
|---|---|---|
| `GAMETYPE_0` | 128 | 128 (unchanged) |
| `GAMETYPE_1` | 64 | **128** |
| `GAMETYPE_2` | 64 | **128** |
| `GAMETYPE_3` | — | 128 (new) |
| `GAMETYPE_4` | — | 128 (new) |
| `GAMETYPE_5` | — | 128 (new) |

### 4. Startup Failure Warning Added to All GAMETYPE_* Descriptions

All `GAMETYPE_*` field descriptions (slots 0–5) now include an explicit warning:

> An invalid game type will cause the server to exit on startup with an error.

This warning was absent from slots 1 and 2 in the previous version. It is now consistent across all six slots.

### 5. Operator Warning Added to Egg Description

The egg's `description` field now opens with a prominent warning about the `/rvs` volume mount requirement before the general game description text:

> IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions.

This warning appears in the panel's egg description field, which operators see before creating a server. It is the earliest feasible point in the setup flow where this requirement can be communicated.

**Note:** This is not a fix for the underlying crash. The hardcoded `/rvs` path is an image-level constraint imposed by the third-party Docker image (`ghcr.io/danpowell88/ravenshield_dedicatedserver`) and cannot be overridden from the egg. This change documents the workaround.

### 6. README: Known Issues Section Added

The README now includes a **Known Issues / Operator Requirements** section with a dedicated subsection for the `/rvs` volume mount requirement. The section documents:

- **Symptoms:** Container crashes immediately on startup with exit code 1; "permission denied" on `/rvs`.
- **Root cause:** `GAMEFILES_DIR` is hardcoded to `/rvs/gamefiles` in the Docker image. Wings mounts server data at `/home/container` by default, which does not satisfy the image's path expectation.
- **Workaround:** Configure the Wings node to bind-mount the server data directory to `/rvs`.
- **Scope clarification:** This is an image-level limitation, not a defect in the egg.

The README also retains a **Known Limitations** section covering the external image dependency, unvalidated map/game type fields, the three-port allocation requirement, and the non-editable entrypoint script.

---

## What Is Not Changed

- Startup command, Docker image reference, install script, config blocks, and stop signal are unchanged.
- Slot 0 (`MAP_0`, `GAMETYPE_0`) descriptions and rules are unchanged.
- All non-rotation variables (`GAME_PRESET`, `PORT`, `NAME`, `MAX_PLAYERS`, `ADMIN_PASSWORD`, `GAME_PASSWORD`, `INTERNET_SERVER`, `INSTALL_OPENRVS`) are unchanged.
- The underlying crash (exit code 1 on `/rvs`) is **not fixed** by this PR. The egg cannot fix an image-level path constraint without modifying the Docker image itself.

---

## Upgrade Notes for Existing Servers

Existing servers using this egg are not affected by the new slots (MAP_3–MAP_5, GAMETYPE_3–GAMETYPE_5 default to blank, which disables those rotation positions). No migration steps are required. Operators who want to use the additional slots may set them after updating the egg definition.

The `max:128` rule change for `GAMETYPE_1` and `GAMETYPE_2` is backwards-compatible — it is a relaxation of the previous constraint, not a tightening.
