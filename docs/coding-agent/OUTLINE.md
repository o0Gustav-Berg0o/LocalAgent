# Course 2: Build a Local Coding Agent


> **Status (final numbering):** the lessons are written. Their numbers match the folders: 01 setup, 02 to 06 Module 2, 07 to 09 Module 3, 10 to 13 Module 4, 14 to 15 Module 5, 16 to 20 Module 6, 21 to 23 Module 7, 24 capstone. Where this outline and the folders differ, the folders are the source of truth. Start at [README.md](README.md).
**Goal:** students build a small, local coding agent in C#, the way a tool like Claude Code works from the inside: a loop, tools, permissions, context management, memory, modes, and frontends.

**Prerequisite:** Course 1 (LocalAgent: notes RAG, tools, eval harness).

**Language:** English. Keep code and output short. Token economy is a design rule (see below).

**Format per lesson:** the same four files as Course 1 (`README.md`, `exercises.md`, `quiz.md`, `review-checklist.md`). Each lesson has one visual or UX item, so the material stays varied.

---

## Two separate things: setup and modes

**First-run setup** happens once, on the first start, and can be run again with `/setup`. It asks questions, scans the machine, suggests a model, and helps install it. It answers "what can this machine run?"

**Settings** are changed at any time with `/settings`: model, effort, permission mode, paths, and output style. They are stored in a file. Setup writes the first values; settings change them later.

**Modes** are a per-session choice, switched with `/mode` or a key:

- **Code mode:** the agent reads, edits, and runs commands with approval rules. Fast, for well-defined tasks.
- **Cowork mode:** the agent writes a plan first, works in steps, and waits for approval at each checkpoint. Slower, for tasks where the user wants to follow along or steer.

A mode is not a setting. Switching modes does not change the model or the permission rules; it changes how the loop asks for approval and when it stops.

---

## Module 0: Setup

### Lesson 0 — First-run setup
- **Build:** a setup wizard. Questions (purpose, patience for waiting, project folder). Read-only hardware scan (RAM, GPU and VRAM, free disk). A model suggestion table by hardware class. Guided install: show the commands and links, and download only after the user confirms the source, file name, and size.
- **Concepts:** read-only scans; why download needs confirmation; first-run versus repeatable setup.
- **Visual:** a checklist that ticks off each setup step; a progress bar during download.
- **Note:** model names and quantisations are checked against current listings when the lesson is written.

---

## Module A: Foundation

### Lesson 1 — The harness, again
- **Build:** the agent loop from Course 1, reused for a coding task (read a file, answer a question about it).
- **Concepts:** messages, tool calls, the loop, the iteration limit.
- **Visual:** streamed tokens as they arrive; a "working" indicator with elapsed seconds.

### Lesson 2 — Local model provider and config file
- **Build:** one provider interface that talks to the local `llama-server`. Endpoint, model name, and context size come from a config file. No keys, no remote endpoints.
- **Concepts:** the single place that knows which model is used.
- **Visual:** a status line showing model name and context size.

### Lesson 3 — Settings and `/settings`
- **Build:** a repeatable settings menu. Model, effort (maps to output length and tool-call limits), permission mode, paths, output style. Values are validated before saving.
- **Concepts:** settings versus setup; why effort is defined per model; validation before saving.
- **Visual:** an arrow-key menu; `/help` with one-line command descriptions; Tab completion for commands.

### Lesson 4 — Read-only tools
- **Build:** read a file, list a folder, search with a `grep`-style tool. Results are truncated with a note saying how much was cut.
- **Concepts:** tool output is input to the model, so it costs tokens; truncation rules.
- **Visual:** tool calls shown as one collapsed line each, expandable with a key.

### Lesson 5 — Permissions
- **Build:** rules for allow, ask, and deny, per tool and per path. Read-only tools are allowed by default. Writes and commands ask.
- **Concepts:** the permission check sits between the model and the tool, never inside the model's prompt.
- **Visual:** the approval prompt with the exact action shown; a badge showing the current permission mode.

---

## Module B: Changing files

### Lesson 6 — Write with approval
- **Build:** a write tool that shows the diff and waits for yes or no. Nothing is written without approval.
- **Concepts:** preview before action; why approval is enforced in code.
- **Visual:** colored diffs (green additions, red removals), with a one-line summary per file.

### Lesson 7 — Edit and undo
- **Build:** an exact-text edit tool (the model gives the old text and the new text). A `/undo` that restores the last change to each file.
- **Concepts:** exact-match edits fail loudly, which is safer than rewriting whole files.
- **Visual:** a change summary after each turn: files touched, lines added and removed.

### Lesson 8 — Shell tool
- **Build:** run a command from an allowlist (build, test, format), with a timeout and a size limit on output.
- **Concepts:** an allowlist, not a blocklist; timeouts; what a runaway command costs.
- **Visual:** a spinner and live output while the command runs; Esc cancels the command, not the session.

---

## Module C: Context and memory

### Lesson 9 — Context budget
- **Build:** count tokens for each message. Show the budget as a percentage. Warn before the limit.
- **Concepts:** the context window is finite; what counts toward it (system prompt, tools, history, tool output).
- **Visual:** a context bar in the status line, colored by how full it is.

### Lesson 10 — Compaction
- **Build:** `/compact` summarises older turns into one message and keeps recent turns verbatim. Automatic compaction at a threshold.
- **Concepts:** what is lost in a summary; why recent turns stay verbatim; compaction costs a model call.
- **Visual:** a notice when compaction runs, showing before and after size.

### Lesson 11 — Project memory
- **Build:** a project instruction file (like CLAUDE.md) loaded into the system prompt. Optional notes the agent writes after a session, which the user can review.
- **Concepts:** memory is text the user can read and edit; keep it short, because it costs tokens every turn.
- **Visual:** `/memory` shows the active files and their sizes, with an open-in-editor option.

### Lesson 12 — Sessions
- **Build:** save a conversation to a file and resume it later, with `/resume` to pick a session.
- **Concepts:** what a saved session contains; secrets never saved.
- **Visual:** a session picker with timestamps, first message, and size.

---

## Module D: Modes and cowork

### Lesson 13 — Code mode and cowork mode
- **Build:** a mode switch. Code mode runs with the permission rules. Cowork mode asks the agent for a plan file first, shows it, and waits for approval before each step. Checkpoints after each step.
- **Concepts:** mode versus settings; the plan as a shared artifact; why checkpoints are per step.
- **Visual:** a mode badge; the plan as a checklist with ticks that update as steps finish.

### Lesson 14 — Shared notes and handoff
- **Build:** a notes file the user writes for the agent, read at the start of each session. The agent writes back what it did and what is left.
- **Concepts:** asynchronous collaboration; the notes file is data, not instructions to obey blindly.
- **Visual:** a notification when the agent is waiting for input or approval.

---

## Module E: Extensibility

### Lesson 15 — Skills
- **Build:** a skill is a folder with a `SKILL.md` file. The model sees only names and descriptions. It loads a skill's full text when the task needs it.
- **Concepts:** lazy loading keeps the prompt small; a description is a prompt too.
- **Visual:** `/skills` lists installed skills, with a short description each.

### Lesson 16 — Custom commands
- **Build:** slash commands defined in files (a name, a description, and a prompt template).
- **Concepts:** repeatable tasks without new code.
- **Visual:** autocomplete that reads command names from the files.

### Lesson 17 — Hooks
- **Build:** run a check after each file change (build or format). Results appear in the same turn.
- **Concepts:** hooks run in code, so they are reliable in a way prompts are not.
- **Visual:** hook results shown inline, with pass and fail marks.

### Lesson 18 — Subagents
- **Build:** a helper that works in its own context and returns a short summary to the main agent.
- **Concepts:** a subagent keeps the main context small; what it can and cannot access.
- **Visual:** a nested progress tree showing the subagent's steps.

### Lesson 19 — External tools (MCP client)
- **Build:** connect to an MCP server and use its tools, under the same permission rules.
- **Concepts:** external tools need the same checks as local ones.
- **Visual:** each tool labelled with the server it came from.

---

## Module F: Frontends and distribution

### Lesson 20 — API separation
- **Build:** the core library exposes events (text, tool call, tool result, question, approval needed) and accepts commands (send, approve, cancel). The console is one frontend.
- **Concepts:** a frontend knows nothing about the loop; the eval harness can drive the core without a console.
- **Visual:** none in this lesson. The focus is the boundary.

### Lesson 21 — Downloading files
- **Build:** a fetch tool with a domain allowlist, a size limit, and a confirmation before anything is saved.
- **Concepts:** untrusted content; saving versus reading.
- **Visual:** a download progress bar.

### Lesson 22 — Packaging and first run
- **Build:** one executable, a README for installing it, and a startup banner.
- **Concepts:** what a user needs on a clean machine.
- **Visual:** a startup banner, theme choice, and plain output when `NO_COLOR` is set.

---

## Capstone — Lesson 23

- **Build:** the agent works on a small sample project. Students run a set of coding tasks (add a function, fix a failing test, rename a symbol) and measure each change with an eval set, as Course 1 does for notes.
- **Deliverable:** a short report with the eval numbers, what changed between versions, and what is still weak.
- **Visual:** a summary table at the end of the run.

---

## Token economy (rules for the course and the app)

- Keep the system prompt short. Move rarely used rules into skills.
- Truncate tool output and say how much was cut.
- Load memory and skills only when relevant.
- Compaction keeps recent turns verbatim, and summarises the rest.
- Students measure tokens per task in the eval, so cost is visible.

## Visual niceties catalogue

Every item appears in one lesson, so the material changes every few pages:

| Item | Lesson |
|---|---|
| Checklist with ticks | 0, 13 |
| Progress bar | 0, 21 |
| Streamed tokens and working indicator | 1 |
| Status line (model, context) | 2, 9 |
| Arrow-key menu, Tab completion, `/help` | 3, 16 |
| Collapsed tool calls | 4 |
| Permission badge and approval prompt | 5 |
| Colored diffs | 6 |
| Change summary | 7 |
| Spinner and cancel key | 8 |
| Context bar | 9 |
| Compaction notice | 10 |
| `/memory` view | 11 |
| Session picker | 12 |
| Mode badge | 13 |
| Notifications | 14 |
| Skills list | 15 |
| Hook pass and fail marks | 17 |
| Nested progress tree | 18 |
| Tool source labels | 19 |
| Startup banner, themes, `NO_COLOR` | 22 |
| Summary table | 23 |

## Open decisions

1. **Model suggestions:** which hardware classes and model sizes go in the table? Check current listings when Lesson 0 is written.
2. **Cowork plan format:** a Markdown checklist, or a structured file the app can parse?
3. **Effort mapping:** how much of "effort" is output length versus tool-call limits? Decide before Lesson 3.
