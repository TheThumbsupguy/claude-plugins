# claude-plugins

TheThumbsupguy's personal [Claude Code](https://claude.com/claude-code) plugin marketplace.

## Install

```
/plugin marketplace add TheThumbsupguy/claude-plugins
/plugin install ttug-sdlc@thethumbsupguy
```

## Plugins

### ttug-sdlc

Agentic SDLC tooling.

- `/ttug-sdlc:build-to-merge` — takes one change from a scope to merged and cleaned up: worktree, build, verify, a real-system check (step 5: what did the tests fake, checked in the cheapest environment that has it for real), review (Claude Code's `/code-review`, which applies the review rules in the repo's CLAUDE.md), PR, merge, then cleanup (worktree, local and remote branch, main pulled; a cloud run leaves the remote branch to the repo's auto-delete). Runs to the end by default; add `ask` to confirm the scope and the merge yourself, `reviews <n>` for more than the default two review rounds, `admin` to merge as admin on green CI with no normal merge tried first, in a repo whose `.claude/build-to-merge.md` has an `Admin merge:` line (ignored elsewhere). Reads a `.claude/build-to-merge.md` in the consuming repo for that repo's own commands, rules and fences (plus an untracked `.claude/build-to-merge.local.md` for one machine's limits), written as `Key: value` lines — the keys are in `skills/build-to-merge/settings.md`; without one it uses defaults. Optional steps: mockups (1b) and a code tour (6b), each switched on in that file.
- `/ttug-sdlc:drive-browser` — drives a real browser page through chrome-devtools-mcp to check a change: your own tab, the cheapest read that answers the question (script, then snapshot, then screenshot), held keys for games, nothing left behind.
- `/ttug-sdlc:device-walk` — walks a change on an Android phone or emulator over adb, checking each step on the phone and in the backend or third party. Plan, run, report, resume; the machine's facts come from a rig file the repo settings name.
- `/ttug-sdlc:video-evidence` — records what a change does on screen (Android device, browser page, Playwright run) and reads it back as numbered frame sheets. Numbers first, video for visual questions. The consuming repo's settings name where videos go.

## Updating

To pick up new versions:

```
/plugin marketplace update thethumbsupguy
```

Versioning is commit-SHA based (no semver yet), so each push is a new version.
