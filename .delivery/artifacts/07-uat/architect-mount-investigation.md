# Architect Investigation: PTDL_v2 Egg Mount Configuration Support

**Date:** 2026-04-11
**Investigator:** Architect agent
**Sources reviewed:** `github.com/pterodactyl/wings` — `server/configuration.go`, `server/mounts.go`, `environment/settings.go`, `environment/docker/container.go`, `config/config.go`

---

## 1. Does the PTDL_v2 egg JSON support a mount field?

**No.** The PTDL_v2 egg JSON has no field for specifying mounts, volumes, or bind-mount paths. The `EggConfiguration` struct in Wings contains only two fields: `id` and `file_denylist`. The egg JSON cannot instruct Wings to bind-mount any path.

---

## 2. Does Wings support per-server bind mounts?

**Yes — but they live in the per-server configuration on the Wings node, not in the egg.**

The `Configuration` struct in `server/configuration.go` includes:

```go
Mounts []Mount `json:"mounts"`
```

This `mounts` array is part of the per-server `server.json` file stored on the Wings node (typically at `/etc/pterodactyl/servers/<uuid>/server.json`). It is pushed to the node by the Panel when server configuration is synced.

The `Mount` type (aliased from `environment.Mount`) has three fields:

```go
type Mount struct {
    Target   string `json:"target"`    // path inside the container
    Source   string `json:"source"`    // host filesystem path
    ReadOnly bool   `json:"read_only"`
}
```

---

## 3. How Wings processes custom mounts at runtime

In `server/mounts.go`, the `Mounts()` method builds the full mount list:

1. **Default mount** (always present, hardcoded):
   ```
   Source: <server data dir on host>
   Target: /home/container
   ReadOnly: false
   ```
2. **Optional `/etc/passwd` and `/etc/group` mounts** (if Wings `config.yml` enables the `passwd` feature).
3. **Custom mounts** from `s.Config().Mounts` — the per-server `mounts` array.

Custom mounts are filtered against `AllowedMounts` in Wings `config.yml`:

```yaml
allowed_mounts:
  - /path/on/host/that/is/allowed
```

A custom mount whose `source` is not prefixed by an entry in `allowed_mounts` is silently skipped with a warning log. This is a **node-level security gate** — the operator must whitelist source paths.

In `environment/docker/container.go`, `convertMounts()` converts all `environment.Mount` entries to Docker bind mounts (`mount.TypeBind`) and passes them to `container.HostConfig.Mounts`.

---

## 4. Would a `/rvs` bind-mount fix the permission crash?

**Yes, exactly.** The mechanism already exists. If the per-server `mounts` array contains:

```json
{
  "target": "/rvs",
  "source": "/path/to/server/data/dir",
  "read_only": false
}
```

Then Wings will bind-mount the server's writable data directory at `/rvs` inside the container **in addition to** the default `/home/container` mount. Since both bind-mount entries point to the same host directory (or the operator can use any whitelisted host path), uid 1000 will have write access to `/rvs`. The `mkdir -p /rvs/setup` and `mkdir -p /rvs/gamefiles` calls in `entrypoint.sh` will succeed, and the permission crash is resolved.

---

## 5. Where is this configured and by whom?

| Layer | What to set | Who sets it |
|---|---|---|
| Wings `config.yml` | Add source path to `allowed_mounts` | Node operator |
| Panel — Server Mounts | Add a mount: source = server data dir (or any whitelisted path), target = `/rvs`, read-only = false | Panel admin |
| Egg JSON (PTDL_v2) | Nothing — no mount field exists | N/A |

The Panel's "Server Mounts" feature (admin panel → Mounts) is the intended UI for this. A mount is created at the node level (with its source path whitelisted in Wings `config.yml`), then assigned to a server. The Panel then syncs the `mounts` array to the Wings node's per-server `server.json`.

---

## 6. Conclusion and recommendation

**The egg JSON cannot specify mounts.** This is a node+panel administration task, not an egg-level concern.

The correct operator workaround is:

1. In Wings `config.yml`, add the intended source directory to `allowed_mounts`.
2. In the Panel admin UI, create a Mount with `target = /rvs` and assign it to the RVS server instance.
3. Wings will then bind-mount a writable path at `/rvs`, eliminating the `Permission denied` crash without any UID changes.

The existing Track B2 recommendation (document this as an operator-side mount workaround in the egg `description`) remains correct. The egg description should explain that a Panel Mount targeting `/rvs` is required until the upstream Docker image is fixed to use `/home/container` or respect a `GAMEFILES_DIR` environment variable.
