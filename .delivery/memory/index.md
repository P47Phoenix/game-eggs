# Memory Index

last_updated: 2026-04-12
runs: 2

## Stage Health (last 5 runs)
| Stage | First-try pass rate | Note |
|-------|--------------------|----|
| Idea | 50% | Architect scope check needed for BUG_FIX on image-layer issues |
| Refine | 50% | QA often finds missing error-case ACs |
| Plan | 100% | |
| Dev | 50% | Description ordering, GAMETYPE invalid-value warning |
| UAT | 100% | |

## Hot Lessons (inject into all agent prompts)
1. docker_images key must equal value = the image URI (not a human label)
2. MAP/GAMETYPE slot 0 must use `required` in rules; slots 1+ can be `nullable`
3. PRD agent may silently change user-specified defaults — cross-check before Checkpoint 1
4. For Wine-based game servers: read the public entrypoint.sh source before writing PRD to confirm env vars, port formula, done string, and map names
5. Shell script baked into Docker image (ENTRYPOINT) is acceptable under repo policy; user-managed wrapper scripts are not
6. Egg description operator warnings must be the VERY FIRST content in the description field
7. GAMETYPE_* descriptions must include invalid-value startup-exit warning for eggs with entrypoint validation

## Topic Pointers
- stages/refine.md — PRD default drift, error-case ACs, Docker policy
- stages/dev.md — docker_images key format, required vs nullable, description ordering, GAMETYPE warnings
- topics/project-types.md — Pterodactyl egg patterns, Wine servers, port formulas, Docker volume semantics
