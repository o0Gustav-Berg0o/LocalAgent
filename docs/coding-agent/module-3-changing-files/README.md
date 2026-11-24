# Module 3: Changing Files Safely

**Goal:** the agent can change files and run commands, and every change is reviewable, reversible, and within limits.

**Why these together:** a write without a review is a risk, a review without undo is incomplete, and a shell without an allowlist is the largest risk of all. The three lessons build on each other in that order.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 07 | [Write with approval and diffs](07-write-approval/README.md) | about 3 hours | propose, review, and apply as three steps, with fingerprints and atomic writes |
| 08 | [Edit and undo](08-edit-undo/README.md) | about 3 hours | exact-text edits, snapshots, and an undo that refuses to overwrite later work |
| 09 | [The shell tool and its allowlist](09-shell-tool/README.md) | about 3 hours | an allowlisted command runner with timeouts, process-tree stops, and output limits |

## Before you start

- Module 2 complete, especially Lesson 06 (permissions) and Lesson 05 (resolved paths).

## What you have at the end

- A write tool that never writes without a passing permission check and an approval.
- Edits that match exactly once, and an undo that checks the file before restoring it.
- Shell commands from an allowlist, run without a shell, with a timeout that stops the whole process tree.
