# Architect Investigation: Container UID Control in Pterodactyl Wings

**Date:** 2026-04-11
**Investigator:** Architect agent
**Sources reviewed:** `github.com/pterodactyl/wings` — `environment/docker/container.go`, `config/config.go`, `server/configuration.go`

---

## 1. Egg-level UID control (PTDL_v2 egg JSON)

**Not possible.**

The `EggConfiguration` struct in Wings (`server/configuration.go` lines 9–17) contains only two fields:

```go
type EggConfiguration struct {
    ID           string   `json:"id"`
    FileDenylist []string `json:"file_denylist"`
}
```

There is no `user`, `uid`, `run_as`, or `features` field. The PTDL_v2 egg JSON format (as consumed by Wings) has no mechanism to override the container user at the egg level. The `features` key present in panel-side egg JSON is a panel UI concern only; it does not pass a UID to Wings.

---

## 2. Wings-level UID control (Wings `config.yml`)

**Possible — but it is a node-wide setting, not per-server.**

Wings resolves the container user in `environment/docker/container.go` (lines 190–194):

```go
if cfg.System.User.Rootless.Enabled {
    conf.User = fmt.Sprintf("%d:%d",
        cfg.System.User.Rootless.ContainerUID,
        cfg.System.User.Rootless.ContainerGID)
} else {
    conf.User = strconv.Itoa(cfg.System.User.Uid) + ":" +
                strconv.Itoa(cfg.System.User.Gid)
}
```

The UID applied to **every** container on the node is whichever of the following resolves first:

| Mode | How UID is determined |
|---|---|
| Normal (default) | UID/GID of the OS user named in `config.yml` `system.username` (typically `pterodactyl`, uid 1000) |
| Rootless (`system.user.rootless.enabled: true`) | `system.user.rootless.container_uid` / `container_gid` from `config.yml` (defaults to 0) |

**To run all containers on a node as root (uid 0):** set `system.user.rootless.enabled: true` and `container_uid: 0` / `container_gid: 0` in Wings `config.yml`. This is the only built-in mechanism.

There is no per-server or per-egg user override. The `server/configuration.go` `Container` struct only exposes `image`; no `user` field is passed from the panel to Wings per-server.

---

## 3. Would a UID change fix the crash?

**Yes, running as uid 0 (root) would fix the permission crash**, because:

- The `/rvs` directory in the Docker image is created by root and owned by root.
- The entrypoint.sh runs `mkdir -p /rvs/setup` and `mkdir -p /rvs/gamefiles` at startup.
- Under uid 1000 these mkdir calls fail immediately (`Permission denied`), causing exit 1.
- Under uid 0 the writes succeed.

However, enabling rootless mode with `container_uid: 0` applies to **all servers on that Wings node**, which is a significant security regression — it removes Pterodactyl's primary container isolation boundary. This is not an acceptable production configuration for a shared node.

The operator options remain:

| Option | UID change needed? | Security impact |
|---|---|---|
| Wings bind-mount: map server data dir onto `/rvs` at node level | No — uid 1000 can write to the bind-mounted path | None (mount is scoped to this server path) |
| Rootless mode with `container_uid: 0` | Yes | High — all containers on node run as root |
| Wait for upstream image fix (`GAMEFILES_DIR` env var or chown in Dockerfile) | No | None |

---

## 4. Impact on the recommended fix (Track B2)

The Wings-level UID change (rootless mode, uid 0) is technically possible but is a node-wide change with serious security implications. It is not a viable recommendation for egg documentation.

**The existing Track B2 recommendation stands unchanged:** document the bind-mount workaround in the egg `description`. The bind-mount approach (operator maps server persistent volume onto `/rvs`) keeps uid 1000 and makes no security trade-offs. It is the correct operator-side workaround until the upstream image is fixed.

The egg description note should additionally clarify that enabling rootless/uid-0 mode is **not recommended** as a workaround because it affects the entire Wings node.
