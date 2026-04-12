# Stage Memory: Dev

## Lesson: docker_images key must equal value (URI, not human label)
validated: 1 | last: run-2026-04-11-r6rs
The PTDL_v2 docker_images field requires key = value = the full image URI (e.g., "ghcr.io/org/image:tag"). Using a human-readable label as the key (e.g., "Game Server") is wrong and will be caught by the Architect DoD validator. Match the pattern in eggs like counter_strike_1.6.

## Lesson: MAP/GAMETYPE slot 0 must be required, not nullable
validated: 2 | last: run-2026-04-12-r6fix
For game server eggs with map rotation variables, MAP_0 and GAMETYPE_0 (the first/mandatory slots) must use `required` in their rules field. MAP_1 onward can be nullable if optional. A nullable MAP_0 allows blank submission that causes server startup failure.

## Lesson: Egg description operator warnings must be FIRST, not appended
validated: 1 | last: run-2026-04-12-r6fix
When an egg has a critical operator requirement (e.g. a non-standard volume mount), the warning must appear as the VERY FIRST content in the description field — not appended after game prose. QA will catch this if it is not first.

## Lesson: GAMETYPE_* descriptions must include invalid-value exit warning
validated: 1 | last: run-2026-04-12-r6fix
For eggs where the entrypoint validates game type values and exits on invalid input, ALL GAMETYPE_* variable descriptions must include a sentence warning that an invalid value causes the server to exit on startup. This is an error-case AC that QA will catch if missing.
