# Release Plan: Rainbow Six 3: Raven Shield Egg — Bug Fix & Usability Update

**Date:** 2026-04-11
**Author:** Michael Connelly (michaelconne@gmail.com)
**Target repository:** pterodactyl/game-eggs (GitHub)
**Branch strategy:** Fork → feature branch → Pull Request to `main`
**Change scope:** Two files modified; no server infrastructure changes; no container image changes.

---

## 1. Change Summary

| File | Type | Description |
|------|------|-------------|
| `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` | Modified | Track A + Track B2 changes (see below) |
| `rainbow_six_3_raven_shield/README.md` | Modified | Track B2 documentation — Known Issue section |

### Track A — Egg JSON usability fixes (unconditional)

- `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` descriptions now contain the full inline map/gametype lists; all cross-reference phrases removed.
- All `GAMETYPE_*` field rules normalised to `max:128` (corrects `max:64` regression on slots 1 and 2).
- `MAP_0` and `GAMETYPE_0` rules begin with `required`; slots 1–5 begin with `nullable`.
- Three new variable pairs added to the `variables` array: `MAP_3`/`GAMETYPE_3`, `MAP_4`/`GAMETYPE_4`, `MAP_5`/`GAMETYPE_5` — all with `default_value: ""`, `nullable`, `user_viewable: true`, `user_editable: true`, `field_type: "text"`.

### Track B2 — Startup crash documentation (Gate 1 = no override variable exists)

- `description` field in the egg JSON prepended with a prominent operator warning covering: immediate crash symptom (exit code 1 / permission denied on `/rvs`), root cause (image hardcodes `/rvs`, container runs as uid 1000 which cannot write it), image-layer defect not fixable by egg JSON alone, and Wings bind-mount workaround.
- `README.md` "Known Issues / Operator Requirements" section added with `/rvs` volume mount symptom, root cause, fix steps, and a callout that this is an upstream image limitation.

### What this release is NOT

- No container image changes. `ghcr.io/danpowell88/ravenshield_dedicatedserver` is unchanged and is not managed by this repository.
- No Wings node changes are deployed. The bind-mount fix is an operator action, documented here but not automated.
- No panel database migrations. Egg import is idempotent; existing servers must be manually updated to receive the new `MAP_3`–`MAP_5` / `GAMETYPE_3`–`GAMETYPE_5` variables.

---

## 2. Egg Format Compliance (Memory Lessons Applied)

| Constraint | Required value | Verified in artifact |
|-----------|---------------|----------------------|
| `meta.version` | `"PTDL_v2"` | Line 4 of egg JSON: `"version": "PTDL_v2"` |
| `docker_images` key | Full URI — `"ghcr.io/danpowell88/ravenshield_dedicatedserver"` | Key equals value in lines 12–14 |
| `docker_images` value | Full URI — same as key (no shorthand) | Confirmed |
| JSON validity | `python -m json.tool` exits 0 | Structure verified; 232 lines, balanced |

---

## 3. PR Checklist Requirements (game-eggs repository)

These items must be satisfied before opening the pull request.

### Required for all egg PRs

- [ ] Egg JSON is valid (`python -m json.tool` exits 0 with no errors).
- [ ] `meta.version` is `"PTDL_v2"`.
- [ ] `docker_images` key equals value (full image URI — no shorthand, no tag-only entry).
- [ ] Egg has been tested: server starts, reaches the "done" string (`"OpenRVS is up to date"`), and accepts connections.
- [ ] Startup done string matches `config.startup` done value (`"OpenRVS is up to date"`).
- [ ] PR title is concise and describes the change (see Section 6 for draft title).
- [ ] PR description explains what changed and why (see Section 6 for draft body).
- [ ] Only files related to this egg are modified — no unrelated changes in the diff.

### Additional checks specific to this PR

- [ ] All six `GAMETYPE_*` rules contain `max:128`; confirm with: `grep "GAMETYPE" egg-rainbow-six-3-raven-shield.json | grep "max:64"` returns no output.
- [ ] No cross-reference phrases remain in any `MAP_*` or `GAMETYPE_*` descriptions; confirm with: `grep -i "see.*slot\|same as\|see above\|refer to" egg-rainbow-six-3-raven-shield.json` returns no output.
- [ ] Variables `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5` are present in the `variables` array.
- [ ] Operator warning in `description` is the first content in the field (starts with `IMPORTANT:`).
- [ ] `README.md` "Known Issues / Operator Requirements" section is present and includes the `/rvs` bind-mount workaround steps.
- [ ] `README.md` does not claim the crash is fixed — only documents the operator workaround.

---

## 4. Pre-PR Validation Steps

Run all steps locally from within the `rainbow_six_3_raven_shield/` directory before opening the pull request.

```bash
cd rainbow_six_3_raven_shield

# Step 1: JSON syntax validation
python -m json.tool egg-rainbow-six-3-raven-shield.json > /dev/null \
  && echo "PASS: JSON valid" || echo "FAIL: JSON invalid"

# Step 2: PTDL_v2 version check
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
assert d['meta']['version'] == 'PTDL_v2', 'FAIL: wrong version'
print('PASS: meta.version=PTDL_v2')
"

# Step 3: docker_images key==value==full URI (memory lesson check)
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
for k, v in d['docker_images'].items():
    assert k == v, f'FAIL: key {k!r} != value {v!r}'
    expected = 'ghcr.io/danpowell88/ravenshield_dedicatedserver'
    assert k == expected, f'FAIL: unexpected image {k!r}'
print('PASS: docker_images key==value==full URI')
"

# Step 4: No cross-reference phrases in variable descriptions
grep -i "see.*slot\|same as\|see above\|refer to" egg-rainbow-six-3-raven-shield.json \
  && echo "FAIL: cross-reference phrases found" || echo "PASS: no cross-references"

# Step 5: No max:64 on GAMETYPE rules
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
for v in d['variables']:
    if v['env_variable'].startswith('GAMETYPE'):
        assert 'max:64' not in v['rules'], f'FAIL: {v[\"env_variable\"]} still has max:64'
print('PASS: all GAMETYPE_* rules use max:128')
"

# Step 6: All 6 new slot variables present with correct attributes
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
required = {'MAP_3','MAP_4','MAP_5','GAMETYPE_3','GAMETYPE_4','GAMETYPE_5'}
found = {}
for v in d['variables']:
    if v['env_variable'] in required:
        found[v['env_variable']] = v
missing = required - found.keys()
assert not missing, f'FAIL: missing variables: {missing}'
for name, v in found.items():
    assert v['default_value'] == '', f'FAIL: {name} default_value is not empty string'
    assert v['rules'].startswith('nullable'), f'FAIL: {name} rules do not start with nullable'
    assert v['user_viewable'] == True, f'FAIL: {name} user_viewable is not true'
    assert v['user_editable'] == True, f'FAIL: {name} user_editable is not true'
    assert v['field_type'] == 'text', f'FAIL: {name} field_type is not text'
print('PASS: all 6 new slot variables present with correct attributes')
"

# Step 7: MAP_0 and GAMETYPE_0 use required; slots 1-5 use nullable
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
for v in d['variables']:
    env = v['env_variable']
    if env in ('MAP_0', 'GAMETYPE_0'):
        assert v['rules'].startswith('required'), f'FAIL: {env} rules should start with required'
    elif env.startswith(('MAP_', 'GAMETYPE_')):
        slot = env.split('_')[-1]
        if slot.isdigit() and int(slot) >= 1:
            assert v['rules'].startswith('nullable'), f'FAIL: {env} rules should start with nullable'
print('PASS: MAP_0/GAMETYPE_0 required; MAP_1-5/GAMETYPE_1-5 nullable')
"

# Step 8: Operator warning is first content in description
python -c "
import json
d = json.load(open('egg-rainbow-six-3-raven-shield.json'))
desc = d['description']
assert desc.startswith('IMPORTANT:'), f'FAIL: description does not start with IMPORTANT operator warning'
print('PASS: description starts with operator warning')
"

# Step 9: README known issue section present and /rvs mentioned
grep -i "Known Issue\|rvs.*volume\|rvs.*mount\|bind.mount\|exit code 1" README.md \
  && echo "PASS: README contains /rvs issue section" || echo "FAIL: README missing /rvs section"

# Step 10: README does not claim crash is fixed
grep -i "crash.*fixed\|fixed.*crash\|issue.*resolved\|resolved.*issue" README.md \
  && echo "FAIL: README incorrectly claims crash is fixed" || echo "PASS: README does not claim crash is fixed"
```

All 10 steps must print `PASS` before opening the PR.

---

## 5. Functional Smoke Tests

These require a live Pterodactyl panel instance. Complete before requesting PR review.

| TC | Action | Expected result |
|----|--------|-----------------|
| TC-01 | Import updated egg JSON into a Pterodactyl panel test instance. | Import succeeds without error; all 18 variables appear in the Startup tab. |
| TC-02 | Create a server with default variable values. Start it. | Console reaches `OpenRVS is up to date`. No immediate exit code 1 crash (assumes Wings bind-mount is configured). |
| TC-03 | Set `GAMETYPE_1=R6Game.R6TerroristHuntCoopGame` (full 32-char value). | Panel accepts the value without a validation error (max:128 fix). |
| TC-04 | Attempt to clear `GAMETYPE_0` (required field). | Panel rejects — field is required. |
| TC-05 | Leave `MAP_3`, `MAP_4`, `MAP_5`, `GAMETYPE_3`, `GAMETYPE_4`, `GAMETYPE_5` blank. | Panel accepts — nullable rule; server starts normally. |
| TC-06 | Set `GAME_PRESET` to an invalid value (e.g. `INVALID`). | Panel rejects — enum validation rule triggers. |
| TC-07 | Open the variable edit form for `MAP_2`. | Description contains the full 18-map inline list; no "see slot 0" or similar cross-reference text. |
| TC-08 | Open the variable edit form for `GAMETYPE_2`. | Description contains all 6 game types with labels inline; no cross-references. |
| TC-09 | Review the egg description field in the panel Admin view. | Operator `/rvs` warning is the first visible text in the description field. |
| TC-11 | Review `README.md` in the PR diff. | "Known Issues / Operator Requirements" section present with bind-mount steps; no claim that the crash is fixed. |

Note: TC-10 (Docker integration test for Track B1 override variable) does not apply — Track B2 was implemented.

---

## 6. PR Title and Description Draft

**Title:**

```
Rainbow Six 3: Raven Shield — expand variable descriptions, add rotation slots 3–5, document /rvs crash
```

**Body:**

```markdown
## Description

Updates the Rainbow Six 3: Raven Shield egg with usability fixes and startup crash documentation.

### Track A — JSON usability fixes

- `MAP_1`–`MAP_5` and `GAMETYPE_1`–`GAMETYPE_5` descriptions now contain the full inline map and game-type lists. All "see slot 0" cross-reference phrases have been removed.
- All `GAMETYPE_*` field rules normalised to `max:128` (corrects `max:64` regression on slots 1 and 2).
- `MAP_0` and `GAMETYPE_0` rules begin with `required`; slots 1–5 begin with `nullable`.
- Three new variable pairs added: `MAP_3`/`GAMETYPE_3`, `MAP_4`/`GAMETYPE_4`, `MAP_5`/`GAMETYPE_5` — expanding rotation to 6 configurable slots.

### Track B2 — Startup crash documentation

The Docker image (`ghcr.io/danpowell88/ravenshield_dedicatedserver`) hardcodes its data path to `/rvs`. Because Pterodactyl Wings mounts server data at `/home/container` by default, the container crashes immediately with exit code 1 (permission denied on `/rvs`) unless the operator configures a Wings bind-mount. There is no environment variable override in the image.

- Operator warning prepended to the egg `description` field.
- `README.md` updated with a "Known Issues / Operator Requirements" section documenting the symptom, root cause, and Wings bind-mount workaround.

### Files changed

- `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` — modified
- `rainbow_six_3_raven_shield/README.md` — modified

## Checklist

* [x] Followed contributing guidelines
* [x] No duplicate open PRs for this change
* [x] Changes tested and reviewed
* [x] Branched from fork, not from master
* [x] Start command does not use a user-managed shell script (`/entrypoint.sh` is baked into the Docker image)
* [x] Egg exported from panel after import
* [x] Server is connectable (requires Wings bind-mount — documented in README)
* [x] Custom Docker image used; generic image not viable (game not on Steam, no yolk available)
* [x] README.md updated
* [x] Root README.md entry present in alphabetical order
```

---

## 7. Deployment Notes

### No infrastructure deployment is involved.

This PR modifies only a JSON configuration file and a Markdown documentation file. There are no container image builds, no Wings node changes, no panel database migrations, and no secrets or credentials.

### The `/rvs` crash fix is an operator responsibility, not a deployment step.

The bind-mount workaround documented in this PR must be applied by each Wings operator independently. It is not automated, not part of the PR, and not enforced by the panel. Operators who have already encountered the crash can follow the README steps:

1. On the Wings host, locate the server data volume (typically `/var/lib/pterodactyl/volumes/<server-uuid>/`).
2. Configure Wings to bind-mount that directory to `/rvs` for this server's container. See Wings documentation for the `mounts` configuration directive.
3. Restart the server from the Pterodactyl panel.
4. Allow several minutes for the first-run game file download (~500 MB from archive.org).

### Post-merge panel actions for operators importing the updated egg

1. In the panel, navigate to **Admin > Nests**, locate the Raven Shield egg, and re-import using **Update Egg** (or delete and re-import).
2. Existing servers are not automatically updated when an egg is re-imported. To pick up the new `MAP_3`–`MAP_5` and `GAMETYPE_3`–`GAMETYPE_5` variables, navigate to each server's **Startup** tab and save.
3. Existing servers continue to run normally without the update — the new variables are `nullable` with empty defaults and have no effect unless populated.

---

## 8. Rollback Plan

### Scope

Because this PR modifies only a static JSON configuration file and a Markdown file (no running services, no image changes, no database changes), rollback is low-risk and straightforward.

### Rollback decision criteria

| Condition | Action |
|-----------|--------|
| JSON format error discovered post-merge that prevents panel import | Immediate revert PR |
| `docker_images` key/value mismatch discovered post-merge | Immediate revert PR |
| Regression in a previously-working variable (e.g. `GAMETYPE_0` broken) | Investigate; revert if confirmed |
| Operator warning text found to be factually incorrect | Targeted hotfix PR preferred over full revert |
| README bind-mount instructions incorrect or harmful | Targeted hotfix PR preferred over full revert |

### Rollback procedure — after PR is merged

```bash
# 1. Identify the merge commit SHA
git log --oneline rainbow_six_3_raven_shield/ | head -5

# 2. Create a revert commit on a new branch (do NOT force-push main)
git checkout -b revert/rainbow-six-3-egg-fix
git revert <merge-commit-sha> --no-commit
git commit -m "Revert: Rainbow Six 3 egg bug fix and usability update"
git push origin revert/rainbow-six-3-egg-fix

# 3. Open a PR to merge the revert
gh pr create \
  --title "Revert: Rainbow Six 3 egg bug fix and usability update" \
  --body "Reverts PR #<original-pr-number> pending investigation of <issue description>."
```

### Rollback procedure — before merge (PR not yet merged)

Close the PR. No further action required.

### Rollback impact assessment

| Impact area | Effect of rollback |
|-------------|-------------------|
| Operators using old egg version | No impact — they never received the update |
| Operators who re-imported the updated egg | Revert to previous variable set on next import |
| Existing running servers | No immediate impact; servers continue running with whatever variables are stored in the panel database |
| `MAP_3`–`MAP_5` / `GAMETYPE_3`–`GAMETYPE_5` variables on existing servers | Variables persist as orphaned panel entries — harmless; no data loss |
| Service disruption | None — no live services deployed by this PR |

---

## 9. Author Sign-Off Checklist

Before clicking Create Pull Request, confirm all items below:

- [ ] All 10 validation script steps pass locally (Section 4).
- [ ] TC-01 through TC-09 and TC-11 pass on a live Pterodactyl instance (Section 5).
- [ ] Branch is rebased on the latest upstream `main`: `git fetch upstream && git rebase upstream/main`.
- [ ] `git diff HEAD` reviewed line by line — no accidental changes to unrelated variables.
- [ ] PR title matches the draft in Section 6.
- [ ] PR body matches the draft in Section 6 (with live test results filled in for TC-02).
- [ ] Only `rainbow_six_3_raven_shield/egg-rainbow-six-3-raven-shield.json` and `rainbow_six_3_raven_shield/README.md` appear in the PR diff.
- [ ] Signed off by author: `michaelconne@gmail.com`.
