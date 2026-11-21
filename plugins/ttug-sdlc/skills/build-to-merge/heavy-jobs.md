<!-- Part of ttug-sdlc:build-to-merge. Read when SKILL.md points here. -->

# Heavy jobs on a shared machine

Applies when a settings file (`Cap:`, `Heavy:`, `Peers:`) or the user's memory names a memory cap or a machine shared with other sessions. A heavy job is a command `Heavy:` lists, or anything as large (a full test suite, a native build, e2e). Otherwise run the command directly.

1. Tell any live peer session what will run and wait for its go. One heavy job at a time.
2. Check free memory covers the cap.
3. Run under the cap, logging to a file outside `/tmp`. The `bash -c` keeps a chained or env-prefixed command wholly inside the cap:
   ```
   systemd-run --user --scope -q -p MemoryMax=<cap> -p MemorySwapMax=0 \
     bash -c '<command>' > <log> 2>&1; echo "exit $?"
   ```
4. Read the result from the log and that exit line. A pipe into `tail` or `head` reports the pipe's exit code, not the command's.
