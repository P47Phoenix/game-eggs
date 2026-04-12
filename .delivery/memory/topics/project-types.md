# Topic Memory: Project Types

## FEATURE — Adding a new Pterodactyl egg (game-eggs repo)
validated: 1 | last: run-2026-04-11-r6rs

Key constraints for this repo:
- Startup command must NOT be a user-managed shell script. Script baked into Docker image is acceptable.
- docker_images key must equal value (both = the image URI)
- PTDL_v2 format required
- README required in game directory + root README update in alphabetical order
- Repo PR checklist requires custom Docker image justification if not using a standard yolk

For eggs using Wine (Windows games via Linux container):
- Check if image has an ENTRYPOINT script — common pattern for Wine-based servers
- Source repo is usually public on GitHub; read entrypoint.sh to confirm valid env vars, port formulas, map/gametype names, and done string before writing PRD
- Port formula may be derived (e.g., PORT / PORT+1000 / PORT+2000) — confirm from source

For map/gametype rotation variables:
- Slot 0 = required (rules: required|string|max:N)
- Slots 1+ = nullable (rules: nullable|string|max:N)

Default value conflicts: user's original specification > PRD agent > test strategy. Always cross-check PRD defaults against user's original message.

For BUG_FIX on Wine/Docker image-layer crashes:
- If the root cause is a hardcoded path in the image (no env var override), split scope into Track A (egg JSON edits, unblocked) and Track B (gated on image maintainer confirmation). Never list "fix crash" as an in-scope egg-JSON deliverable when the root cause is image-layer.
- `RUN chown -R 1000:1000 /path` in a child image (`FROM upstream`) CANNOT fix permissions on a path declared as `VOLUME` in the parent — Docker discards the chown layer at volume init. A working fix requires a full independent image rebuild, not a `FROM upstream` extension.
- Docker anonymous volumes inherit permissions from the image layer at the point the VOLUME was declared. Any RUN commands above that point in a child image are no-ops for that path.
