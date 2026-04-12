# Dev Log — STORY-01: Rainbow Six 3: Raven Shield Egg Improvements

**Date:** 2026-04-11
**Developer:** Claude (delivery-team:developer)
**Story:** STORY-01

---

## Summary

Implemented all Track A and Track B2 items for the Rainbow Six 3: Raven Shield Pterodactyl egg (prior sprint). This revision log covers the two blocking QA findings (F-01, F-02) identified in the QA DoD review dated 2026-04-11.

---

## QA Revision — Blocking Fixes Applied

### F-01 (BLOCKING) — AC-B2-02: Operator warning moved to first position in description

**Problem:** The egg `description` field began with game prose ("Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter..."), with the IMPORTANT /rvs operator warning appearing after approximately 2 sentences. AC-B2-02 requires the warning to be the very first content in the description field.

**Fix Applied:** Restructured the `description` field so the IMPORTANT /rvs warning paragraph appears first, followed by the game prose and OpenRVS context. No warning text was altered — only the ordering was changed.

**Before (start of description):**
> "Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter released in 2003... IMPORTANT: This image requires the game data volume to be mounted at /rvs..."

**After (start of description):**
> "IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions. Tom Clancy's Rainbow Six 3: Raven Shield is a tactical first-person shooter..."

---

### F-02 (BLOCKING) — AC-A-29: Invalid game type warning added to all GAMETYPE_* descriptions

**Problem:** All 6 GAMETYPE_* variable descriptions (GAMETYPE_0 through GAMETYPE_5) listed valid values with human-readable labels but contained no warning that supplying an invalid value causes the entrypoint to exit with an error before the server launches.

**Fix Applied:** Appended the sentence "An invalid game type will cause the server to exit on startup with an error." to the end of the description for GAMETYPE_0, GAMETYPE_1, GAMETYPE_2, GAMETYPE_3, GAMETYPE_4, and GAMETYPE_5.

**Slots updated:** All 6 (GAMETYPE_0 through GAMETYPE_5).

**Appended text:** `An invalid game type will cause the server to exit on startup with an error.`

---

## Revision Verification

- JSON validity confirmed: `python -m json.tool` exits 0.
- No other variables, startup command, or structural elements were modified.
- GAMETYPE_* rules (required/nullable, max:128) unchanged.
- MAP_* variables unchanged.
- All unmodified variables (GAME_PRESET, PORT, NAME, MAX_PLAYERS, ADMIN_PASSWORD, GAME_PASSWORD, INTERNET_SERVER, INSTALL_OPENRVS) intact.

---

## Files Modified

- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` — two targeted edits as described above.

---

## Prior Sprint Changes (for reference)

### egg-rainbow-six-3-raven-shield.json

**Track A — Map/Gametype Rotation Expansion (Tasks 1–5)**

1. **Added MAP_3, MAP_4, MAP_5** — new nullable map rotation slots with `nullable|string|max:64` rules and empty string defaults.
2. **Added GAMETYPE_3, GAMETYPE_4, GAMETYPE_5** — new nullable gametype slots with `nullable|string|max:128` rules and empty string defaults.
3. **Updated MAP_1 and MAP_2 descriptions** — replaced "See MAP_0 for valid map names" with the full inline map list verbatim.
4. **Updated GAMETYPE_1 and GAMETYPE_2 descriptions** — replaced "See GAMETYPE_0 for valid values" with the full inline gametype list including labels.
5. **Fixed GAMETYPE_1 and GAMETYPE_2 rules** — changed `max:64` to `max:128` to normalise all GAMETYPE_* slots.

**Track B2 — Description and README Update (Tasks 6–7)**

6. **Updated egg `description` field** — appended the operator note about the `/rvs` volume mount requirement.
7. **README** — Added "Known Issues / Operator Requirements" section and updated Environment Variables table.

---

## Verification Checklist

- [x] JSON is valid (`python -m json.tool` exits 0)
- [x] docker_images key == value == full URI
- [x] MAP_0: `required|string|max:64`
- [x] GAMETYPE_0: `required|string|max:128`
- [x] MAP_1–MAP_5: `nullable|string|max:64`
- [x] GAMETYPE_1–GAMETYPE_5: `nullable|string|max:128`
- [x] All field_type values are "text"
- [x] All gametype slot descriptions include full inline gametype list with labels
- [x] All gametype slot descriptions now include invalid-value startup exit warning (F-02 fix)
- [x] Egg description begins with IMPORTANT /rvs operator warning (F-01 fix)
- [x] No changes to: startup, config, scripts, author, name, meta, exported_at, or unrelated variables

---

## Second Revision — Panel Mounts Language Update (2026-04-11)

**Task:** Replace vague "configure your Wings server to bind-mount..." language with specific Pterodactyl Panel Mounts instructions.

### egg-rainbow-six-3-raven-shield.json — description field

**Before (warning text):**
> "IMPORTANT: This image requires the game data volume to be mounted at /rvs. Pterodactyl Wings mounts server data at /home/container by default, which causes the container to crash on startup with exit code 1 (permission denied on /rvs). To fix this, configure your Wings server to bind-mount the server data directory to /rvs instead of /home/container. See the README for full instructions."

**After (warning text):**
> "IMPORTANT: This image writes game data to /rvs (hardcoded). Pterodactyl mounts data at /home/container by default, causing an immediate startup crash (exit code 1). Fix: create a Panel Mount (Admin → Mounts) with Target = /rvs and assign it to this server. See the README for full steps."

### README.md — Known Issues / /rvs Volume Mount Requirement

**Before (Fix paragraph):**
> "Configure your Wings node to bind-mount the server data directory to /rvs instead of /home/container. This is done at the Wings server level, not in the Pterodactyl panel egg configuration."

**After:** Replaced with 5-step numbered procedure using correct Panel Mounts terminology:
1. Admin → Mounts — create Mount with Source (host path), Target `/rvs`, Read Only: No
2. Assign Mount to this Egg and the Node
3. Open server in admin panel → Mounts → click + to assign to server instance
4. Add source path to `allowed_mounts` in Wings `config.yml`; restart Wings
5. Start the server

### Verification
- JSON validity confirmed: structure unchanged, only description string edited.
- README: only the Fix paragraph in the Known Issues section was changed; all other content preserved.
- Warning remains first content in the description field.
