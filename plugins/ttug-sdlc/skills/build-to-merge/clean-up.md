<!-- Part of ttug-sdlc:build-to-merge. Read when SKILL.md points here. -->

# Clean up (step 9)

This is the step that gets skipped. Run all of it, in order, as soon as the PR is merged.

1. Confirm the merge landed: `gh pr view <PR> --json state,headRefOid`. `state` must read `MERGED`; anything else, stop. (A PR closed unmerged gets this cleanup only on the owner's word that the branch is not needed.)
2. Confirm nothing is unpushed: `git rev-parse <branch>` must equal that `headRefOid`. If it differs, the branch holds commits the PR never had: stop and ask.
3. Stop every dev server, watcher and background job this change started.
4. In the worktree: `git status --porcelain --ignored`. Tracked or untracked changes: stop and ask. Ignored files that exist nowhere else (env files, build folders, generated notes) are deleted with the worktree, silently: copy them to the main checkout or ask first.
5. Remove the worktree. **Native** first, where the tool exists: `ExitWorktree({ action: "remove", discard_changes: true })`. It acts only on a worktree this session made with `EnterWorktree` and is still in (step 8 stays there); for any other, a resumed session included, it reports no worktree session and does nothing: go to Manual. After a squash merge it always refuses without `discard_changes`, since `<branch>` holds commits the session's starting commit lacks; steps 1, 2 and 4 are what make that safe, so never pass it before they pass. It deletes the folder and `worktree-<name>`, keeps `<branch>`, and moves the session back to the main checkout. Native only checks against the session's start, never the PR head, which is why steps 1, 2 and 4 stay. **Manual**, everywhere else (a headless run, a subagent, a worktree made by hand or by an earlier session): `cd "$MAIN" && git -C "$MAIN" worktree remove <path>` in one call, so the shell is not inside the folder it removes (a subagent's shell returns to the worktree path on every call, so give every later command `-C "$MAIN"`); if git says it is locked, `git -C "$MAIN" worktree unlock <path>` first. Never `--force` past tracked changes. Desktop's clean-up of inactive worktrees is not this step: it takes only sessions idle for 30 days or more, keeps their branches, and is unavailable over WSL.
6. `git -C "$MAIN" branch -D <branch>`, and `worktree-<name>` if it is still there. Steps 1 and 2 are what make the force safe; `-d` is no guide here, since it compares against the branch's upstream, not the default branch.
7. Delete the remote branch if the merge left it: `gh api -X DELETE repos/<owner>/<repo>/git/refs/heads/<branch>`. A ref delete pushes no code, so it does not need the pre-push hook that `git push --delete` would run. In a cloud session, skip it: the branch goes as SKILL.md's Cloud session says.
8. Main checkout on `<default>`: `git -C "$MAIN" pull --ff-only --prune`. On another branch: `git -C "$MAIN" fetch --prune origin <default>:<default>`, and say so.
9. Update whatever record the repo file names (checkpoint, retro, follow-ups, release notes).

Final check, and report its output:

```
git -C "$MAIN" worktree list --porcelain | grep -Fx -e 'branch refs/heads/<branch>' -e 'branch refs/heads/worktree-<name>'
git -C "$MAIN" branch --list '<branch>' 'worktree-<name>'
git -C "$MAIN" ls-remote origin 'refs/heads/<branch>'
ls -d <worktree path>
```

The first three print nothing and `ls` reports no such directory: clean. In a cloud session, an `ls-remote` line left by the merge goes in the final report, not retried.
