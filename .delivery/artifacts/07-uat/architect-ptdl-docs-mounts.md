# Architect Investigation: Pterodactyl Official Docs — Mounts Feature

**Date:** 2026-04-11
**Sources:** pterodactyl.io/guides/mounts.html, pterodactyl.io/community/config/eggs/creating_a_custom_egg.html, pterodactyl.io/wings/1.0/configuration.html

---

## What Mounts Are (official definition)

The official docs describe mounts as a feature that permits administrators to **"mount other directories from the host file-system into a Server's container."** This enables shared access to host system directories within containerized server environments.

Mounts are a **Panel admin concept** — they are created and managed by administrators, not by end-users or eggs.

---

## How to Create a Mount in the Panel Admin UI

1. Navigate to **Mounts** in the admin Panel.
2. Create a new mount entry with these fields:
   - **Name**: identifier
   - **Description**: details
   - **Source**: "The absolute path to the folder or files on the Node machine"
   - **Target**: "The absolute path where the mount will be placed inside of your server"
   - **Read Only**: access restriction toggle
   - **User Mountable**: whether users can self-mount
3. Assign applicable **Eggs** and **Nodes** to the mount definition.

---

## How to Assign a Mount to a Server

1. Open the target server in the admin Panel.
2. Navigate to the server's **Mounts** page.
3. Click the **+** button to add the mount.
4. Restart the server.

The mounted content then becomes accessible at the target path inside the container.

---

## Wings `allowed_mounts` Configuration Key

In `/etc/pterodactyl/config.yml`, the `allowed_mounts` field explicitly lists host directories that are permitted for mounting. All subdirectories of listed paths are also permitted. Wings must be restarted after any changes to this file.

This is a **node-level security gate**: if a mount's source path is not covered by an entry in `allowed_mounts`, Wings silently skips it (confirmed by source code review in prior investigation).

---

## Can an Egg JSON Define or Reference a Mount?

**No.** The official egg-authoring documentation (community guide) contains no mention of mounts, volumes, or bind-mount fields. The PTDL_v2 egg JSON format has no mechanism to define or reference a mount. This is confirmed by both:
- The official docs (no mount fields described in egg authoring guide), and
- Wings source code review (prior artifact): `EggConfiguration` struct contains only `id` and `file_denylist`.

---

## Key Finding: Mounts Always Require Manual Operator Steps

The official Pterodactyl docs make clear that mounts are a **Panel + Wings administration task** with no egg-level automation:

| Step | Who does it | Where |
|---|---|---|
| Add source path to `allowed_mounts` | Node operator | Wings `config.yml` |
| Create Mount definition (source, target) | Panel admin | Admin UI → Mounts |
| Assign Eggs and Nodes to the mount | Panel admin | Admin UI → Mounts |
| Assign mount to a specific server instance | Panel admin | Admin UI → Server → Mounts |
| Restart server | Admin or user | Panel |

**There is no way to encode a mount in the egg JSON so that it is automatically configured when the egg is imported.** Operators must always perform the Panel + Wings configuration steps manually.

---

## Conclusion

The official docs fully corroborate the Wings source code findings from the prior investigation. The mount system is entirely outside the egg's control. Any egg that requires a non-standard bind mount (e.g., `/rvs`) must document this as an operator prerequisite in the egg description. The egg cannot automate it.
