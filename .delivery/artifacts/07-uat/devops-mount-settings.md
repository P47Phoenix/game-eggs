# DevOps: Panel Mount Settings for Rainbow Six 3: Raven Shield

**Date:** 2026-04-11
**Role:** Operations

---

## Panel Mount Field Values

| Field | Value |
|---|---|
| **Name** | `rvs-gamefiles` |
| **Description** | Maps the server data directory to /rvs so the RVS Docker image can write game files to /rvs/gamefiles and /rvs/setup |
| **Source** (host path) | `/var/lib/pterodactyl/volumes/<server-uuid>` — see note below |
| **Target** (container path) | `/rvs` |
| **Read Only** | No |
| **User Mountable** | No — this mount is required for the server to function; users should not be able to detach or modify it |

---

## Finding the Server UUID (Source Path)

The server UUID is visible in the Panel admin UI:

1. Go to **Admin → Servers** and open the target server.
2. The UUID is shown in the server detail page (typically in the URL or the server info section).
3. The full host path is: `/var/lib/pterodactyl/volumes/<that-uuid>`

Alternatively, on the Wings node run:
```
ls /var/lib/pterodactyl/volumes/
```
Each directory name is a server UUID. Match it to the server by cross-referencing the Panel.

---

## Wings `config.yml` — `allowed_mounts`

**Yes, the parent directory is sufficient.** You do not need to list each server UUID path individually. Adding the parent covers all servers:

```yaml
allowed_mounts:
  - /var/lib/pterodactyl/volumes/
```

Wings checks that the mount `source` is prefixed by an entry in `allowed_mounts`. The parent directory `/var/lib/pterodactyl/volumes/` matches all UUID subdirectories, so this one entry covers every server on the node. Restart Wings after editing `config.yml`.

---

## Why This Works

Wings always bind-mounts the server data directory at `/home/container`. By adding a second mount pointing the same host directory at `/rvs`, both paths inside the container resolve to the same host location. Game files written to `/rvs/gamefiles` or `/rvs/setup` are stored in the server's allocated data directory — no separate host directory is needed.

---

## Summary Checklist

- [ ] Add `/var/lib/pterodactyl/volumes/` to `allowed_mounts` in Wings `config.yml`
- [ ] Restart Wings (`systemctl restart wings`)
- [ ] Create the Mount in Panel admin with the values above
- [ ] Assign the Mount to the Raven Shield egg and the relevant Node
- [ ] Assign the Mount to the specific server instance (Admin → Server → Mounts → click +)
- [ ] Restart the server
