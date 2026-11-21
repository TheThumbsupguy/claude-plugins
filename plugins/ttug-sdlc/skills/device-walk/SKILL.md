---
name: device-walk
description: Validates a change on a real Android phone or emulator, driven end to end by the agent over adb — every changed user path walked on the device, with the external systems (store, payments, server) read and changed by API so each step is checked on both sides. Plans the walk from the change's acceptance criteria, runs it, records evidence, resumes. Use for a build-to-merge "check for real" step on a mobile app, after a feature is built or merged and before calling it done, or when the owner says "walk it on the phone". Not a substitute for the test suites.
argument-hint: [plan|run|resume] <record folder, PR number, or feature name>
---

# Device walk

The agent drives everything: the phone, the backend, and any third party the feature talks to. A person only unlocks what the agent cannot (a device port, a password-manager session) and rules on anything that writes to production.

## 0. Load the rig

The rig is one file of this machine's facts for this repo. The repo's untracked `.claude/build-to-merge.local.md` names its path as `Rig: <path>`; read it from the main checkout (a worktree does not have it), or use the copy build-to-merge already read. Read it first, and the memories it links when their row matters. No rig, or the rig does not cover something: stop and ask. Never guess a device, a backend or a credential route. When a walk learns something, update the rig (§6).

## 1. Plan — `plan`

1. **Scope from the change, not from memory.** Read the issue's or spec's acceptance criteria, the PR body and the diff. List every user path the change adds or alters, plus paths it depends on that no device has proven yet (search earlier walks and reviews; say what you searched).
2. **Drop what is already proven** on a device, citing where. Mark what this rig cannot drive and name the suite that covers it instead.
3. **Pick the backend per phase** from the rig. A shared backend has merged code only if something deploys it: never hand-push unmerged migrations or functions to it. Production is read-only unless the owner rules on a named step.
4. **Write `DEVICE-WALK.md` in the record folder** the rig names: a rig summary, a read-only preflight, one table per phase with a row per path — **path · agent does · expect on the phone · expect in the external system or server · AC** — then teardown and a "not walked, and why" list. A row that only looks at the screen is incomplete.
5. **Name test data** so it can be found and torn down (`DW <date> <n>`), and plan its teardown.

Show the owner the phases and the "not walked" list before a phase that writes anywhere shared.

## 2. Run — `run`

1. **Preflight** exactly as written; any failure stops the phase.
2. **Build and install** per the rig. Prove the install by the package's `lastUpdateTime` (`dumpsys package`), never by the installer's exit code.
3. **Per step:** act on the phone → `screencap` → read the external system and the server → record the row in `<record folder>/evidence/device/<date>/RESULTS.md`, never in the worktree (pass or fail · screenshot path · the readings). A screenshot is the truth when a `uiautomator dump` disagrees. A step whose result unfolds over time (submit → loading → navigation, a sync, a reported flash) gets a video, since screenshots miss a screen that lasts under 2 s; it is read as frame sheets with `ttug-sdlc:video-evidence`; the repo settings name where videos go.
4. **On a failure:** stop the phase, keep the state, report it with its evidence. Never re-run a step until it passes without saying so: a flaky pass is a finding.
5. **Teardown** as planned, the video on the phone included; list anything that could not be removed.

## 3. Driving rules

- Tap at coordinates read off a fresh `uiautomator dump`; re-read after every keyboard or screen change. Press-and-hold is `input swipe x y x y 1500`.
- No blind BACK or ESC: either can close a custom tab or sheet.
- Type with `input text` (spaces as `%s`); never focus-type into a window that could take a keystroke as a command.
- Secrets travel in process memory only — piped from the password manager, never echoed, logged or written to a file the transcript can see.
- Personal data shown on screen stays out of published evidence.

## 4. Report

One table: phase · steps · pass · fail · not walked. Link `RESULTS.md` and the evidence folder. Each failure: the step, what was expected, what the screenshot and readings show.

## 5. Resume — `resume`

Read `RESULTS.md`, re-run the preflight, continue from the first step that failed or has no result.

## 6. Writing a rig

| Section | Answers |
| :-- | :-- |
| Device | Which phone or emulator, how to connect (and what a person must read off the device), the adb binary |
| Build | How to build an installable app per backend (env, command, output path, package id), and which build types work on this machine |
| Accounts | Test accounts per backend and what each may do |
| Backends | How to read (and, where allowed, write) each backend; which writes need the owner |
| External systems | How to reach each third party (API, credential route, test mode) |
| Record + evidence | The record folder for `DEVICE-WALK.md` and `evidence/`, frame-sheet `tile` |
| Not drivable | Paths this rig cannot drive, and why |
| Traps | Known false greens and hazards, each with its check |
