# Retrospective — Rainbow Six 3: Raven Shield Pterodactyl Egg

**Date:** 2026-04-11
**Project type:** New Pterodactyl egg — game-eggs repo
**Pipeline stages run:** Idea → Refine → Plan → Dev → UAT
**Stages skipped:** Design, Architect
**Total DoD rounds:** Idea=1, Refine=2, Plan=2, Dev=2, UAT=2
**Self-corrections across pipeline:** 6

---

## What Went Well

- **Escalation resolved by reading source, not guessing.** When the Refine stage hit the no-shell-script policy question (the Docker image uses `/entrypoint.sh`), the team went straight to the public GitHub source repo to verify the entrypoint is baked into the image — not a user-supplied script. Resolved in one loop with no ambiguity.
- **Port formula confirmed from authoritative source.** `PORT+1000` (query) and `PORT+2000` (VoIP) were read directly from `entrypoint.sh` rather than assumed, giving high confidence the egg wires ports correctly.
- **All game data confirmed from source.** The full gametype enum, valid map names, and the server "done" startup string were all verified from the actual game server codebase, not documentation or guesswork.
- **No licensing blocker.** Game files auto-download from archive.org — no CD key, no DRM, no takedown risk. Confirmed early so the team never wasted effort on workarounds.
- **All artifacts landed correctly.** JSON is valid, egg file is on disk at the right path, and the root `README.md` was updated to include the new entry.

---

## What Didn't Go Well

- **PRD agent silently changed user-specified defaults.** `NAME` and `MAX_PLAYERS` were set by the user in the original brief. The PRD agent substituted different values without flagging the divergence. This wasn't caught until UAT PO review — two stages too late.
- **QA test strategy invented its own defaults.** At different stages, the QA strategy used `"Pterodactyl Raven Shield"` and `MAX_PLAYERS=16` — values that matched neither the user spec nor the PRD at those points. This created a moving target for pass/fail comparisons.
- **`docker_images` key format not caught until Architect DoD round 2.** The field requires a URI as the key (e.g., `ghcr.io/...`), not a human-readable label. The correct format was not listed in the idea brief constraints, so it wasn't on anyone's checklist until late.
- **`MAP_2` default conflict persisted across stages.** A known discrepancy between the PRD and user stories went unresolved from Plan through most of UAT before being explicitly reconciled. "Known but unresolved" is not a valid state to carry forward.

---

## Improvement Actions

1. **PRD agent must flag default-value divergences explicitly.**
   Whenever the PRD agent sets a default that differs from what the user specified in the original brief, it must call out the divergence in a clearly marked note (e.g., `> NOTE: User specified X; PRD sets Y — confirm intent`). This is a required output, not optional commentary.

2. **QA test strategy must derive defaults from the current approved PRD.**
   Before writing expected values into any test case, the QA agent must pull the canonical defaults from the PRD artifact on disk. Invented or remembered values are not acceptable. If the PRD lacks a default for a given variable, QA must raise a blocker, not assume.

3. **Add `docker_images` field format to the idea brief template for egg projects.**
   The constraint — key must be the full image URI, not a human label — should be in the checklist that every new egg idea brief inherits. This prevents the format error from reaching Dev or Architect review.

---

## One Thing to Remember Next Time

**When adding a new Pterodactyl egg: read the entrypoint source before writing a single variable.**
The Docker image's `entrypoint.sh` is the ground truth for startup string, port offsets, environment variable names, and game-specific enum values. Every hour spent reading that file saves two hours of DoD corrections later. Do it at Refine, document what you find, and treat it as the authoritative spec for Dev — not the game's own documentation, not similar eggs, and not assumptions.
