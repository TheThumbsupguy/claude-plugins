<!-- Part of ttug-sdlc:build-to-merge. Read when SKILL.md points here. -->

# Mockups (step 1b)

Step 1b, on only when the repo file switches it on (with the artboard size and where an approved canvas is saved). It is a fence: it waits for the owner. A change with no UI says "no UI, skipped".

- Before any branch or code: a Design canvas (Artifact `quickstart`, intent `design`; without that tool, stop and ask), one static artboard per screen or state, titled with an id (`U1`, `U2`…), no clickable flow. Colours, fonts and radii come from the repo's theme.
- Draw every state up front: list each situation the screen looks different in (viewer, editor, loading, empty, error, another account's data) and draw them all.
- The owner approves each by id (`approve U2`, `amend U3: …`); build only what is approved.
- A state that appears later (a review fix, a device walk) is new UI even when it reuses an approved look: draw it as a new id on the same canvas. That is a fence on the merge: add `Fence: merge waits for approval of U<n>` to the PR body.
- Keep the canvas; put its link in the PR body.
