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

**Module 0: Foundations (Course 1)**
Read these before Module 1. They are the basis for everything in this course.
- [Agents, tools and the loop](../01-agents-tools-loop/README.md)
- [Embeddings and semantic search](../02-embeddings-semantic-search/README.md)
- [Chunking strategies](../03-chunking-strategies/README.md)
- [Retrieval, grounding and hallucination](../04-retrieval-grounding-hallucination/README.md)
- [Tool design for small models](../05-tool-design-small-models/README.md)
- [Context window and memory](../06-context-window-memory/README.md)
- [Prompt injection and confirmed writes](../07-prompt-injection-confirmed-write/README.md)
- [Eval harness](../08-eval-harness/README.md)
- [Hybrid search](../09-hybrid-search/README.md)

**Module 1: Setup**
- [01 First-run setup](01-first-run-setup/README.md)

**Module 2: Foundations of the agent**
- [02 The harness, again: streaming and feedback](02-harness-streaming/README.md)
- [03 Local model provider and config file](03-provider-config/README.md)
- [04 Settings and the `/settings` menu](04-settings-menu/README.md)
- [05 Read-only tools: read, list, search](05-read-tools/README.md)
- [06 Permissions: allow, ask, deny](06-permissions/README.md)

**Module 3: Changing files safely**
- [07 Write with approval and diffs](07-write-approval/README.md)
- [08 Edit and undo](08-edit-undo/README.md)
- [09 The shell tool and its allowlist](09-shell-tool/README.md)

**Module 4: Context and memory**
- [10 Context budget](10-context-budget/README.md)
- [11 Compaction](11-compaction/README.md)
- [12 Project memory](12-project-memory/README.md)
- [13 Sessions](13-sessions/README.md)

**Module 5: Modes and cowork**
- [14 Code mode and cowork mode](14-modes-cowork/README.md)
- [15 Shared notes and handoff](15-shared-notes/README.md)

**Module 6: Extensibility**
- [16 Skills](16-skills/README.md)
- [17 Custom commands](17-custom-commands/README.md)
- [18 Hooks](18-hooks/README.md)
- [19 Subagents](19-subagents/README.md)
- [20 External tools (MCP client)](20-mcp-client/README.md)

**Module 7: Frontends and distribution**
- [21 API separation: core and frontends](21-api-separation/README.md)
- [22 Downloading files safely](22-downloads/README.md)
- [23 Packaging and first run](23-packaging/README.md)

**Capstone**
- [24 Capstone: the agent on a sample project, measured](24-capstone/README.md)

## Time

Each lesson is planned for about three hours: reading, lab, and reflection. Module 1 and the capstone are longer. Plan a weekend for each module if you work evenings.

## Rules that apply throughout

- **Token economy:** keep the system prompt short, truncate tool output and say how much was cut, and load memory and skills only when relevant.
- **Consent before action:** downloads, writes, and commands need a yes from the user. Reading and scanning do not.
- **No secrets in files:** the config file and memory files hold non-secret values only.
- **Measure every change:** each lesson that changes behaviour adds to the eval set or runs it, so you can see the effect in numbers.
