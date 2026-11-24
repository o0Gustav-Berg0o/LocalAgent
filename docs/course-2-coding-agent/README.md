# Course 2: Build a Local Coding Agent

A hands-on course in which you build a small, local coding agent in C#, from an empty console to a packaged tool. The agent works the way a tool like Claude Code works on the inside: a loop, tools, permissions, context management, memory, modes, and frontends. Everything runs on your own machine, with models served by `llama-server`.

**Prerequisite:** Course 1 (LocalAgent: notes search, tools, eval harness). Module 0 below links the Course 1 lessons you need.

**Language:** English.

## What you will build

By the end you will have a console agent that:

- Sets itself up on first start: asks questions, scans the machine, suggests a model, and guides the installation. Nothing is downloaded without your explicit yes.
- Reads and searches files, edits them with your approval, and runs an allowlist of commands.
- Manages its context window: shows how full it is, and compacts older turns when needed.
- Remembers things across sessions through a project memory file and saved sessions.
- Switches between **code mode** (act with permission rules) and **cowork mode** (plan first, work in steps, wait for approval).
- Extends through skills, custom commands, hooks, subagents, and external MCP tools.
- Separates its core from its frontend, so the console is one of several possible interfaces.
- Can be packaged for another person to install.

## How a lesson is built

Every lesson folder has the same four files:

| File | Purpose |
|---|---|
| `README.md` | The concept, the reasoning behind it, and the gotchas. Read it fully before the lab. |
| `exercises.md` | Step-by-step lab with checkpoints, observe questions, and discussion. Long on purpose. |
| `quiz.md` | Questions with an answer key and a scoring guide. |
| `review-checklist.md` | Things you can check, not things you believe. |

Each lesson has one visual or user-experience item, so the work stays concrete. The list is in [OUTLINE.md](OUTLINE.md).

## Module map

Each module is a folder. It has an overview (`README.md`) and its lessons inside.

- **[Module 0: Foundations (Course 1)](module-0-foundations/README.md)**: read the Course 1 lessons first.
- **[Module 1: Setup](module-1-setup/README.md)**
  - [01 First-run setup](module-1-setup/01-first-run-setup/README.md)
- **[Module 2: Agent foundations](module-2-agent-foundations/README.md)**
  - [02 The harness, again: streaming and feedback](module-2-agent-foundations/02-harness-streaming/README.md)
  - [03 Local model provider and config file](module-2-agent-foundations/03-provider-config/README.md)
  - [04 Settings and the `/settings` menu](module-2-agent-foundations/04-settings-menu/README.md)
  - [05 Read-only tools: read, list, search](module-2-agent-foundations/05-read-tools/README.md)
  - [06 Permissions: allow, ask, deny](module-2-agent-foundations/06-permissions/README.md)
- **[Module 3: Changing files safely](module-3-changing-files/README.md)**
  - [07 Write with approval and diffs](module-3-changing-files/07-write-approval/README.md)
  - [08 Edit and undo](module-3-changing-files/08-edit-undo/README.md)
  - [09 The shell tool and its allowlist](module-3-changing-files/09-shell-tool/README.md)
- **[Module 4: Context and memory](module-4-context-memory/README.md)**
  - [10 Context budget](module-4-context-memory/10-context-budget/README.md)
  - [11 Compaction](module-4-context-memory/11-compaction/README.md)
  - [12 Project memory](module-4-context-memory/12-project-memory/README.md)
  - [13 Sessions](module-4-context-memory/13-sessions/README.md)
- **[Module 5: Modes and cowork](module-5-modes-cowork/README.md)**
  - [14 Code mode and cowork mode](module-5-modes-cowork/14-modes-cowork/README.md)
  - [15 Shared notes and handoff](module-5-modes-cowork/15-shared-notes/README.md)
- **[Module 6: Extensibility](module-6-extensibility/README.md)**
  - [16 Skills](module-6-extensibility/16-skills/README.md)
  - [17 Custom commands](module-6-extensibility/17-custom-commands/README.md)
  - [18 Hooks](module-6-extensibility/18-hooks/README.md)
  - [19 Subagents](module-6-extensibility/19-subagents/README.md)
  - [20 External tools (MCP client)](module-6-extensibility/20-mcp-client/README.md)
- **[Module 7: Frontends and distribution](module-7-frontends-distribution/README.md)**
  - [21 API separation: core and frontends](module-7-frontends-distribution/21-api-separation/README.md)
  - [22 Downloading files safely](module-7-frontends-distribution/22-downloads/README.md)
  - [23 Packaging and first run](module-7-frontends-distribution/23-packaging/README.md)
- **[Module 8: Capstone](module-8-capstone/README.md)**
  - [24 Capstone: the agent on a sample project, measured](module-8-capstone/24-capstone/README.md)

## Time

Each lesson is planned for about three hours: reading, lab, and reflection. Module 1 and the capstone are longer. Plan a weekend for each module if you work evenings.

## Rules that apply throughout

- **Token economy:** keep the system prompt short, truncate tool output and say how much was cut, and load memory and skills only when relevant.
- **Consent before action:** downloads, writes, and commands need a yes from the user. Reading and scanning do not.
- **No secrets in files:** the config file and memory files hold non-secret values only.
- **Measure every change:** each lesson that changes behaviour adds to the eval set or runs it, so you can see the effect in numbers.
