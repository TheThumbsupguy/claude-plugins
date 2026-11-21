<!-- Part of ttug-sdlc:build-to-merge. Read when SKILL.md points here. -->

# Code tour (step 6b)

Step 6b, on only when the repo file switches it on (`Code tour: on`, optionally with a place outside the worktree to save tours). It helps the owner understand the code; it is not another review.

Write it after the last review round, on the code as it will merge. It is a fence, like Mockups: show it to the owner and wait for their word before step 7 — done (`tour done`) or skip (`skip tour`). Resumed before the PR with no record of the word: show the tour again and wait. A change with no code path to follow (docs, prose) carries `Code tour: no code path, skipped` in the PR body; unsure, write the tour.

- **Intent first:** two plain sentences — what the change does, and what it leaves out on purpose. Name anything the scope asked for that was not built.
- **Stops:** up to 11, about 5 for a typical change, titled with an id (`C1`, `C2`…). Reading order along the main path, not file order: where the user's action enters, each layer it passes through, then the edge cases. Changed files, plus at most two unchanged ones the path cannot be followed without. Ids are never reused or renumbered: a stop added later takes the next free id.
- **Each stop:** a file:line checked on the branch, never guessed, and four short lines — what this is, how it works, why it matters, what a careful reader would miss. Plain words; a code name only with its meaning beside it.
- **Risk:** mark the stops that touch money, auth, stored data or code other places call.
- **Questions:** two or three (`Q1`…), each naming its stop, at least one "if this broke, where would you look first?" Not a test: a missed answer gets that stop explained again, and the owner's next reply counts as answered.
- **Replies:** the owner may reply by id (`got C2`, `ask C3: …`, an answer to `Q1`); answer each ask before moving on. Nothing is tracked per stop: the owner's one word releases the fence.
- **After a change:** an ask that turns up a defect is a fix — step 4 again and a review round. Any code change after the tour (that fix, a CI fix) re-checks every stop's file:line, adds stops where needed (with `gh pr edit` when the tour is in the PR body), and updates the rounds line. A change before the word: show the updated stops again. A change after it updates the tour; the word stands. A defect found after the merge: report it; a new call starts the fix.
- **Where:** shown in the chat, then at step 7 put in the PR body, verbatim, and checked verbatim on read-back. A place the repo file names, outside the worktree: save it there, show the owner the link, and link it from the PR body.
