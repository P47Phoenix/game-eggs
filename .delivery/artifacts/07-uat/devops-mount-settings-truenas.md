# DevOps: Panel Mount Follow-Up — TrueNAS SCALE

**Date:** 2026-04-11
**Role:** Operations
**Context:** Follow-up to `devops-mount-settings.md` — answers two admin questions.

---

## Q1: How Do I Assign the Mount to a Server in the Panel?

Navigate to the server's admin page and attach the mount there:

1. **Admin → Servers** — find and open the target Raven Shield server.
2. In the server detail page, click the **Mounts** tab (top navigation within the server admin view).
3. The tab lists all mounts that are eligible for this server (i.e., mounts assigned to the same Node). Find `rvs-gamefiles` and click the **+** (attach) button next to it.
4. The mount is now assigned. Restart the server for the bind-mount to take effect.

**Prerequisite:** The mount must already be assigned to the correct **Node** (done when creating the mount: Admin → Mounts → Edit → Nodes tab). If `rvs-gamefiles` does not appear on the server's Mounts tab, go to Admin → Mounts → `rvs-gamefiles` → Nodes and add the node that hosts this server.

---

## Q2: TrueNAS SCALE — What Host Paths to Use

**The path depends on your Wings installation method.** The two most common cases:

### Case A: TrueCharts Pterodactyl / Wings app (most common)

TrueCharts runs Wings in a container. The Wings data directory is mounted into that container from a TrueNAS dataset, typically:

```
/mnt/<pool-name>/ix-applications/releases/wings/volumes/<pvc-name>/
```

The `volumes/` subfolder inside that PVC is where Wings stores server data — equivalent to `/var/lib/pterodactyl/volumes/` on standard Linux.

To find the exact path, run on the TrueNAS host:

```bash
find /mnt -path "*/pterodactyl/volumes" -type d 2>/dev/null
# or
find /mnt -path "*/wings*volumes*" -type d 2>/dev/null
```

Once found (example result: `/mnt/tank/ix-applications/releases/wings/volumes/pterodactyl-volumes-pvc-xxxx/volumes/`), use that as your base.

### Case B: Wings installed directly on TrueNAS (bare-metal / jail / VM)

If Wings was installed directly (e.g., in a Linux VM or Debian jail), the path is the standard:

```
/var/lib/pterodactyl/volumes/
```

---

### Field Values for TrueNAS SCALE (Case A)

| Field | Value |
|---|---|
| **Source** (Panel Mount) | `/mnt/<pool>/ix-applications/releases/wings/volumes/<pvc-name>/volumes/<server-uuid>` |
| **allowed_mounts** (Wings config.yml) | `/mnt/<pool>/ix-applications/releases/wings/volumes/<pvc-name>/volumes/` |

Use the parent `volumes/` directory in `allowed_mounts` — this covers all server UUIDs with one entry.

---

### Where is Wings `config.yml` on TrueNAS SCALE?

- **TrueCharts app:** The config is typically stored in the PVC alongside the volumes data, or mounted from a hostPath configured during app setup. Check the TrueCharts app config in the TrueNAS UI under **Apps → wings → Edit** to see the declared config path. A common location is:
  ```
  /mnt/<pool>/ix-applications/releases/wings/volumes/<config-pvc-name>/etc/pterodactyl/config.yml
  ```
  You can also locate it by running:
  ```bash
  find /mnt -name "config.yml" -path "*/pterodactyl/*" 2>/dev/null
  ```

- **Direct install (Case B):** Standard path applies:
  ```
  /etc/pterodactyl/config.yml
  ```

After editing `config.yml` to add `allowed_mounts`, restart Wings (TrueCharts: restart the app via the TrueNAS UI or `k3s kubectl rollout restart` for the Wings deployment; direct install: `systemctl restart wings`).
