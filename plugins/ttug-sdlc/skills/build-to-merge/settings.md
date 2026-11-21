<!-- Part of ttug-sdlc:build-to-merge. The settings format every repo uses. -->

# Settings format

Each settings file is a list of `Key: value` lines, one per setting, holding only what differs from the defaults. A value names the tool to use when there is one. A value too long for one line points to a `## <Heading>` further down the same file, where the long form lives. To write a fence, end the value in `fence: <who>`. Some keys are fences by themselves: `Merge: ask`, `Mockups: on` and `Code tour: on`. What a fence does is in SKILL.md's stop table.

```
# Build to merge — <repo>

Code tour: on
Verify: yarn verify
Check for real (gameplay): play-test after the PR opens · fence: <who> · see Play-test
Merge: ask

## Play-test
<the long form>
```

## Keys — `.claude/build-to-merge.md` (the repo's rules)

Every ttug-sdlc skill reads its settings from these files; this is the one list of keys. The Step column names the build-to-merge step, or the skill that reads the key.

| Key | Step | Example | Default |
| :-- | :-- | :-- | :-- |
| `Scope:` | 1 | `a GitHub issue with a label` | an issue, or the owner's own words |
| `Mockups:` | 1b | `on · 960×540 · canvas saved to <path>` | off |
| `Worktree:` | 2 | `~/dev/<repo>-wt-<name>, branch feat/<slug>` | in-repo `.claude/worktrees/<name>` |
| `Build:` | 3 | `spec and code commits separate` | the repo's own commit style |
| `Verify:` | 4 | `yarn verify` | a `verify` script, else lint + typecheck + tests |
| `Check for real (<type>):` | 5 | `Check for real (UI): device walk on staging` | none |
| `Reviews:` | 6 | `3` | 2 (a repo may set 1, never 0; a call's `reviews <n>` can raise it) |
| `Code tour:` | 6b | `on` | off |
| `PR:` | 7 | `draft` | ready for review |
| `CI:` | 7 | `wait 5 minutes for checks` | read from workflow files |
| `Merge:` | 8 | `ask`, or `second approver · fence: <who>` | straight after green CI |
| `Admin merge:` | 8 | `allowed on <owner>'s word · required checks: validate, audit` | not allowed |
| `Record:` | 9 | `checkpoint updated` | none |
| `Fence:` | any | `step 3 waits for <who>` | none |
| `Environments:` | 5 | `see Environments` (each one: what it has for real, what it lacks, how to reach it; cheapest first; and which steps need the owner: shared backends, secrets, production writes) | none: report what the tests faked, touch nothing |
| `Readout:` | drive-browser, video-evidence | `window.__game.debug()` | none: read the page |

## Keys — `.claude/build-to-merge.local.md` (this repo on this machine) and `~/.claude/build-to-merge.local.md` (every repo on this machine)

| Key | Example |
| :-- | :-- |
| `Browser:` | `chrome-profile start e2e 9222` (drive-browser: the launcher and its port) |
| `Videos:` | `~/.cache/<repo>-logs/<branch>/videos/` (video-evidence: raw videos only; frame sheets go where the repo keeps evidence) |
| `Cap:` | `MemoryMax=12G per heavy command` |
| `Heavy:` | `yarn e2e, yarn build` |
| `Ports:` | `5173, 8081` |
| `Logs:` | `~/.cache/<repo>-logs/<branch>/` |
| `Peers:` | `ask live sessions before a heavy job` |
| `Rig:` | `<path to the device-walk rig file>` |
