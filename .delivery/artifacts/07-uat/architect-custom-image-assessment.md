# Architect Assessment: Custom Docker Image to Fix /rvs Permissions

**Date:** 2026-04-11
**Author:** Architect agent
**Context:** Evaluating a custom image extending `ghcr.io/danpowell88/ravenshield_dedicatedserver` as an alternative to the Track B2 bind-mount workaround.

---

## Question 1: Does `chown -R 1000:1000 /rvs` in a child image fix anonymous volume permissions?

**No. The chown will be silently discarded at runtime.**

Docker's volume layer semantics mean the `VOLUME` directive in the *parent* image is inherited by the child. When a `VOLUME` instruction appears in any ancestor layer, Docker records that path as a volume mount point. On first container start, Docker initialises the anonymous volume by copying the image's filesystem content at that path — but it copies from the *image layer that declared the VOLUME*, not from layers added afterward.

A `RUN chown -R 1000:1000 /rvs` placed *after* the inherited `VOLUME` in the child Dockerfile adds a new layer, but that layer is **above** the volume declaration. Docker's volume initialisation snapshot is taken from the layer state *at the point the VOLUME was declared*. The chown layer is never reflected in the initialised volume. The directory is initialised as root-owned, exactly as in the parent.

This is a well-known Docker layering trap: `RUN` commands on paths already declared as `VOLUME` in an ancestor are no-ops from a persistence perspective.

---

## Question 2: Is there a Dockerfile pattern that reliably makes /rvs writable by uid 1000?

**Yes — but it requires modifying or rebuilding from scratch, not extending.**

The only reliable patterns are:

| Pattern | Works? | Notes |
|---|---|---|
| `RUN chown` *after* inherited `VOLUME` | No | Chown layer discarded at volume init |
| Re-declare `VOLUME /rvs` after `RUN chown` in the *same* Dockerfile | No | VOLUME can only be declared once per path; re-declaring is a no-op in a child |
| **Build a fully independent image** (not `FROM upstream`) that reproduces the game server setup, omits `VOLUME /rvs`, and adds `RUN chown` | **Yes** | Complete ownership of the Dockerfile; no inherited volume |
| Replace the entrypoint to `chown /rvs` at container start (before entrypoint.sh runs) | **Yes** | Requires a wrapper entrypoint that runs as root, chowns, then `exec su-exec 1000 entrypoint.sh`; needs `su-exec`/`gosu` in the image |

The two-line `FROM upstream + RUN chown` Dockerfile proposed by the product owner does **not** work due to the inherited volume semantics described above.

---

## Question 3: What would a working custom image require?

A viable custom image would need to be a **full independent build** (not a child of the upstream):

```dockerfile
# Must reproduce all upstream setup steps — not FROM upstream
# Install game server dependencies, Steam, RavenShield...
# Do NOT declare VOLUME /rvs (or declare it after chown)
RUN mkdir -p /rvs && chown -R 1000:1000 /rvs
# Copy a modified entrypoint.sh that honours a GAMEFILES_DIR env var
COPY entrypoint.sh /entrypoint.sh
USER 1000
ENTRYPOINT ["/entrypoint.sh"]
```

**Hosting:** A new container registry (GitHub Packages GHCR, Docker Hub, or a self-hosted registry) separate from the upstream. The game-eggs repo does not host images.

**Maintenance burden:** High.
- The upstream image must be monitored for base OS, Steam, or game server updates.
- Every upstream change requires a manual rebuild and re-tag.
- The egg's `docker_images` key must be updated to pin the new image URI on each rebuild.
- If the upstream image is abandoned, this image becomes the de-facto upstream with full support obligations.

---

## Question 4: Does game-eggs repo policy allow custom images?

**Yes, with justification.** The repo PR checklist requires explicit justification when an egg's `docker_images` value is not a standard yolk image (i.e., not under `ghcr.io/pterodactyl/yolks`). A custom image is permitted but must:

- Be documented in the egg description or a linked README.
- Have the `docker_images` key set to the exact full image URI of the custom image.
- Include a justification comment in the PR explaining why a standard yolk is insufficient.

This is not a policy blocker, but it adds PR review friction.

---

## Question 5: Is this better than Track B2 (bind-mount docs)?

**No. Track B2 is the better near-term fix.**

| Criterion | Custom Image | Track B2 (bind-mount docs) |
|---|---|---|
| Actually solves the permission problem | Only if rebuilt from scratch (not FROM upstream) | Yes — bind-mount makes /rvs writable by uid 1000 |
| Complexity | High (full Dockerfile rewrite + registry hosting) | Low (operator config, one-time) |
| Security impact | None if done correctly | None |
| Maintenance burden | High (track upstream forever) | None |
| Dependency on upstream | Reduced (own image) | Dependent on upstream image |
| Risk of breakage on upstream update | High (diverge silently) | Low |
| Time to ship | Days to weeks | Hours |

The custom image approach as proposed (two-line `FROM + RUN chown`) is technically broken due to Docker volume inheritance semantics. The corrected approach (full independent rebuild) is viable but carries disproportionate long-term maintenance cost for a single game's permission quirk.

---

## Recommendation

**Reject the two-line custom image proposal as technically incorrect.** A `RUN chown` in a child image cannot override a `VOLUME` declared in the parent — the permission change is discarded at volume initialisation.

**Ship Track B2.** The bind-mount workaround is correct, secure, zero-maintenance, and ships immediately. If the upstream image ever adds a `GAMEFILES_DIR` env var or a proper `chown` before its `VOLUME` declaration, the workaround can be retired with a one-line egg update.

If the team later decides a custom image is worth the maintenance investment, it must be a full independent rebuild — not a `FROM upstream` child — and must be accompanied by a hosted registry and an ongoing image maintenance commitment.
