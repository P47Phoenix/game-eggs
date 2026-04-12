# Product Owner DoD Review — Stage 1 Idea Brief

**Brief:** Rainbow Six 3: Raven Shield — Greenfield Pterodactyl Egg
**Reviewer role:** Product Owner
**Review date:** 2026-04-12
**Pipeline run:** run-2026-04-12-r6egg
**Artifact reviewed:** `.delivery/artifacts/01-idea/po/idea-brief.md`
**Review version:** 1

---

## Gate Criteria Assessment

### 1. Problem Statement Is Clear and Specific — PASS

The problem statement identifies a concrete, reproducible failure: the existing egg's reliance on `ghcr.io/danpowell88/ravenshield_dedicatedserver` hardcodes game data to `/rvs`, which conflicts with Pterodactyl's non-root UID model and `/home/container` mount point. The symptom is explicit (`mkdir: cannot create directory '/rvs/setup': Permission denied`). The PO directive is unambiguous: abandon the upstream image and build a proper egg from scratch using a standard base image with Wine runtime.

---

### 2. Target Users Are Identified — PASS

Target users are explicitly named: "Pterodactyl panel operators" who want to run Raven Shield dedicated servers without custom Docker images, Panel Mounts, or Wings config changes. The audience is specific and operationally scoped.

---

### 3. Goals Are Measurable and Achievable — PASS

Five numbered goals are stated. Each is discrete and testable:
1. Egg starts successfully on any standard Pterodactyl + Wings installation (binary pass/fail)
2. Uses a standard base image — no custom Docker builds (inspectable)
3. Install script handles game file download into `/home/container` (structural)
4. Startup command launches via Wine from `/home/container` (observable)
5. All user-configurable variables retained: MAP_0-5, GAMETYPE_0-5, GAME_PRESET, PORT, NAME, MAX_PLAYERS, passwords, INTERNET_SERVER, INSTALL_OPENRVS (JSON field count)

All goals are achievable given the reference material and prior pipeline findings cited in the brief.

---

### 4. Constraints Are Listed — PASS

Six constraints are enumerated, covering format (PTDL_v2), infrastructure requirements (no Panel Mounts, no custom Docker image), data location (`/home/container`), environment compatibility (containerized Wings / TrueNAS SCALE), and upstream reference repo. Each is binary-verifiable and imposes a clear design boundary.

---

### 5. Initial Scope Is Defined with Clear Boundaries — PASS

The "Initial Scope" section lists seven discrete deliverables:
- Analyze upstream Dockerfile/entrypoint.sh
- Select public base Docker image
- Write egg install script
- Write startup command
- Create full egg JSON with variables, validation rules, descriptions
- Create README
- Test on user's TrueNAS SCALE instance

Each item maps to a concrete output artifact. The scope is bounded to creating a single new egg — no infrastructure changes, no upstream contributions.

---

### 6. Out of Scope Is Defined — PASS

Four items are explicitly excluded:
- Modifying or forking the upstream Docker image
- Contributing fixes upstream
- Supporting other game variants/mods beyond base Raven Shield + OpenRVS
- Panel Mount documentation

These exclusions directly address likely scope-creep vectors and clearly delineate where work stops.

---

### 7. Brief Provides Enough Context for Downstream Stages — PASS

**Refine/Design stage:** The reference material section (bottom of brief) provides confirmed technical details from the prior `run-2026-04-12-r6fix` pipeline: 18 valid map names enumerated, 6 valid game types with full class paths, port formula, done string, entrypoint.sh loop behavior (MAP_0-31, GAMETYPE_0-31), and download sources (archive.org, GitHub releases). This eliminates research ambiguity for downstream agents.

**Architect stage:** Constraints explicitly name the target format (PTDL_v2), the data path (`/home/container`), the runtime stack (Wine + Xvfb on Ubuntu base), and the reference repo for extracting install/launch logic. The architect has clear technical guardrails.

**Plan/Dev stage:** The scope items map directly to development tasks. Variable names are enumerated (MAP_0-5, GAMETYPE_0-5, etc.), and the brief specifies slot counts (6 map, 6 gametype) — bounded and unambiguous.

**Memory lessons applied:**
- `docker_images` key must equal value (the image URI) — the brief specifies "use existing public base images" which downstream must resolve to a concrete URI; this is appropriate at Stage 1 level.
- MAP/GAMETYPE slot 0 must use `required`; slots 1+ can be `nullable` — the brief references MAP_0-5 and GAMETYPE_0-5, consistent with this rule. Downstream design/architect stages will enforce the validation rule detail.
- Egg description operator warnings must be the VERY FIRST content — not explicitly stated in the brief, but this is a design-level concern appropriately handled downstream.
- GAMETYPE_* descriptions must include invalid-value startup-exit warning — same as above; appropriate for Design stage.

---

## Summary Table

| Criterion | Result |
|-----------|--------|
| Problem statement clear and specific | PASS |
| Target users identified | PASS |
| Goals measurable and achievable | PASS |
| Constraints listed | PASS |
| Initial scope defined with clear boundaries | PASS |
| Out of scope defined | PASS |
| Brief provides enough context for downstream stages | PASS |

**ALL criteria PASS.**

---

## Overall Verdict: APPROVED — Proceed to Stage 2 (Refine)

The Stage 1 idea brief is well-formed for a GREENFIELD project. The problem is clearly motivated by a confirmed production failure, the solution direction (build from scratch with standard base image) is explicit, and the reference material from the prior BUG_FIX pipeline provides confirmed technical details that eliminate research ambiguity for downstream stages. No blocking issues identified.
