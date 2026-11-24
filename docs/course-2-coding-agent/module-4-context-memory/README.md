# Module 4: Context and Memory

**Goal:** the agent stays useful over a long session. It knows how full its context is, it can shrink a conversation without losing what matters, it remembers the project between sessions, and it can resume where it stopped.

**Why these together:** the budget shows the problem, compaction fixes it, memory keeps the facts that compaction must not lose, and sessions keep the record that all three depend on.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 10 | [Context budget](10-context-budget/README.md) | about 3 hours | a meter for the five parts of a request, a budget with a reserve, thresholds, and a refusal before sending |
| 11 | [Compaction](11-compaction/README.md) | about 3 hours | placeholdering first, then a summary with fixed sections, with the transcript kept on disk |
| 12 | [Project memory](12-project-memory/README.md) | about 3 hours | a user-written instruction file, approved agent notes, a size limit, and a `/memory` view |
| 13 | [Sessions](13-sessions/README.md) | about 3 hours | append-only session files, a validated resume, a picker, and deletion with confirmation |

## Before you start

- Module 2 complete (the loop, the provider, settings, and read tools).
- Module 3, Lesson 09 (the shell output that fills the context fastest).

## What you have at the end

- A status line that shows how full the context is, with the numbers.
- A conversation that can be shortened on purpose, with the original kept on disk.
- Project facts that survive restarts, each one approved and visible.
- Sessions that resume from a validated record.
