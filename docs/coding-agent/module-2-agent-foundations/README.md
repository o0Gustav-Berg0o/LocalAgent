# Module 2: Agent Foundations

**Goal:** turn the Course 1 loop into an agent with a proper foundation: streaming output, a provider behind one interface, settings the user can change, read-only tools, and the first permission rules.

**Why these together:** every later module depends on the loop streaming complete messages, on the provider being swappable, on settings that persist, and on one place that decides what an action may do.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 02 | [The harness, again: streaming and feedback](02-harness-streaming/README.md) | about 3 hours | streamed answers, a working indicator, Esc to cancel, and two timing numbers per turn |
| 03 | [Local model provider and config file](03-provider-config/README.md) | about 3 hours | one provider interface, every server value in the config, validation on load |
| 04 | [Settings and the `/settings` menu](04-settings-menu/README.md) | about 3 hours | a settings menu with validation before save, effort levels, atomic saves |
| 05 | [Read-only tools: read, list, search](05-read-tools/README.md) | about 3 hours | three read tools with path checks, output limits, and binary detection |
| 06 | [Permissions: allow, ask, deny](06-permissions/README.md) | about 3 hours | the rule engine, modes as presets, the ask prompt, and the decision log |

## Before you start

- Module 1 complete (setup and config).
- Course 1, Lesson 1 (the loop) and Lesson 7 (confirmed writes).

## What you have at the end

- An agent that streams its answers, can be cancelled, and reports timing.
- A provider that the loop cannot see past, with a fake provider for tests.
- A settings menu whose every change is validated and saved safely.
- Read tools that cannot leave the project and cannot flood the context.
- One permission check that every tool call passes through.
