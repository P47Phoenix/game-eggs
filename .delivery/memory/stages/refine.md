# Stage Memory: Refine

## Lesson: PRD agent changes user-specified defaults without flagging
validated: 1 | last: run-2026-04-11-r6rs
When the PRD agent sets default_value for env vars, it may silently diverge from the user's original spec. Always cross-check PRD defaults against the user's original requirements message before approving Checkpoint 1.

## Lesson: Error-case ACs are easy to miss
validated: 1 | last: run-2026-04-11-r6rs
The QA DoD validator caught that PRDs for CLI/server config eggs need explicit ACs for invalid input (e.g., invalid GAME_PRESET exits with error). Add this as a standard AC for any egg with an enum env var.

## Lesson: Read source before escalating on Docker image policy
validated: 1 | last: run-2026-04-11-r6rs
When an egg uses a custom Docker image with a shell script ENTRYPOINT, check if the script is baked into the image (acceptable) vs user-managed (not acceptable). The public GitHub source repo is the fastest way to verify this.
