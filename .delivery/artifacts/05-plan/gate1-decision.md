# Gate 1 Decision: GAMEFILES_DIR Configurability

**Date:** 2026-04-11  
**Investigator:** Architect agent  
**Source reviewed:** https://github.com/danpowell88/ravenshield_dedicatedserver  
**Decision:** Track B2

---

## Evidence

### entrypoint.sh — lines 4–5 (hardcoded assignment)

```bash
GAMEFILES_DIR="/rvs/gamefiles"
SETUP_DIR="/rvs/setup"
```

These are plain shell variable assignments. There is no `${GAMEFILES_DIR:-/rvs/gamefiles}` pattern, no `export GAMEFILES_DIR` that reads from the environment first, and no conditional logic that would allow an operator-supplied value to override them. The variables are set unconditionally to literal strings.

Searched the entire entrypoint.sh for any environment variable fallback pattern affecting `GAMEFILES_DIR` or `SETUP_DIR` — none found. The path `/rvs` is referenced directly in all subsequent operations (mkdir, unzip, cp, crudini, cd, wine launch).

### Dockerfile — ENV declarations

```dockerfile
ENV DISPLAY=:0.0
ENV INI_CFG=RavenShield.ini
ENV SERVER_CFG=Server.ini
ENV INSTALL_OPENRVS=true
ENV PATCH_R6GAMESERVICE=true
ENV OPENRVS_SERVER_INFO_INTERVAL=300
```

No `ENV GAMEFILES_DIR` declaration. No `ENV DATA_DIR`. No `ENV RVS_DIR` or any equivalent that would allow an operator to redirect the data directory path at container runtime.

### Summary of search

| Variable | Configurable via env? | Evidence |
|---|---|---|
| `GAMEFILES_DIR` | No | Hardcoded `"/rvs/gamefiles"` — no `:-` fallback, no Dockerfile ENV |
| `SETUP_DIR` | No | Hardcoded `"/rvs/setup"` — no `:-` fallback, no Dockerfile ENV |
| `INI_CFG` | Yes (Dockerfile ENV) | `RavenShield.ini` default, overridable |
| `SERVER_CFG` | Yes (Dockerfile ENV) | `Server.ini` default, overridable |

---

## Gate 1 Decision: Track B2

Track B1 (add `GAMEFILES_DIR` env var to the egg) is **not viable**.

The image's entrypoint.sh hardcodes `GAMEFILES_DIR="/rvs/gamefiles"` with no environment variable override mechanism. Passing a `GAMEFILES_DIR` env var into the container at runtime has no effect — the entrypoint overwrites the variable value unconditionally on lines 4–5 before using it. No egg-level change can redirect the data write path.

Track B2 is the correct fix: add an operator guidance note to the egg `description` field. The note must explain:

1. The Docker image writes all game data to `/rvs/gamefiles` and `/rvs/setup`, which are hardcoded in the image's entrypoint.sh.
2. Pterodactyl mounts the server's persistent data volume at `/home/container`, not `/rvs`.
3. The container will crash immediately (exit 1, `mkdir: cannot create directory '/rvs/setup': Permission denied`) when run under Pterodactyl's default uid 1000 user without a custom Wings bind-mount of `/rvs`.
4. This is an image-layer defect. It cannot be resolved by egg JSON changes alone. A custom Wings volume configuration or infrastructure-level bind-mount of `/rvs` is required.
5. Users who want a working server must either wait for the image maintainer to add a configurable data directory path, or configure their Wings node to bind-mount the server data directory onto `/rvs` — which is outside standard Pterodactyl panel configuration.

**Success Criteria 1–3 from the idea brief are removed from this PR's definition of done.** The PR delivers Track A only (description improvements, map/gametype slot expansion). The startup crash is documented as an image-layer defect.

---

## Recommendation for egg description note (draft)

> **Important — Startup crash known issue:** This egg uses the image `ghcr.io/danpowell88/ravenshield_dedicatedserver`, which hardcodes its data directory to `/rvs/gamefiles`. Pterodactyl mounts server data at `/home/container`, not `/rvs`. The container will crash immediately on startup (Permission denied on `/rvs`) unless your Wings node is configured to bind-mount the server data directory onto `/rvs`. This is an image-layer defect that cannot be resolved by egg configuration changes. See https://github.com/danpowell88/ravenshield_dedicatedserver for upstream tracking.
