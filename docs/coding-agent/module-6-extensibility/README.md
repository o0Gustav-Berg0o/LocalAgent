# Module 6: Extensibility

**Goal:** the agent can be extended without changing its core. Skills give it know-how that loads on demand. Custom commands save repeated requests. Hooks run checks reliably. Subagents work in their own context. External tools come from MCP servers.

**Why these together:** each one adds a new source of text or action to the agent. Each one is therefore checked by the same rules, and none of them may change those rules.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 16 | [Skills](16-skills/README.md) | about 3 hours | skill folders with a short index, and a `load_skill` tool that loads bodies on demand |
| 17 | [Custom commands](17-custom-commands/README.md) | about 3 hours | command files that expand to a user message, with help and autocomplete |
| 18 | [Hooks](18-hooks/README.md) | about 3 hours | user-written checks that run at events, with a file the model cannot change |
| 19 | [Subagents](19-subagents/README.md) | about 3 hours | a separate context with its own tool list and budget, returning one result |
| 20 | [External tools (MCP client)](20-mcp-client/README.md) | about 3 hours | connections to MCP servers, prefixed names, rules that apply to every external tool |

## Before you start

- Module 5 complete.
- Module 2, Lesson 06 (permissions) and Module 3, Lesson 09 (the process rules that hooks and servers reuse).

## What you have at the end

- Skills and commands that extend the agent without touching its code.
- Hooks that run a check every time, from a file only the user can write.
- Subagents that keep the main context small, under the same rules.
- External tools that are checked the same way as local ones.
