# Module 5: Modes and Cowork

**Goal:** the agent works in two modes. Code mode acts within the permission rules for a task the user understands. Cowork mode plans first, works in gated steps, and leaves a handoff the user can read later.

**Why these together:** a mode changes the gate, and the handoff is what makes a gate useful when the user is away.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 14 | [Code mode and cowork mode](14-modes-cowork/README.md) | about 3 hours | the mode switch, a plan file with three parsing rules, approval gates, checkpoints, and rollback |
| 15 | [Shared notes and handoff](15-shared-notes/README.md) | about 3 hours | a user-written notes file read as data, an agent-written handoff built from the record, and staleness warnings |

## Before you start

- Module 4 complete (sessions and the transcript are needed for the handoff).
- Module 3 complete (approval, undo, and the shell rules are used inside each step).

## What you have at the end

- A session that asks you to approve a plan before any step runs.
- A checkpoint after each step, and a rollback that refuses if files changed.
- A handoff file that lists only what the record shows.
