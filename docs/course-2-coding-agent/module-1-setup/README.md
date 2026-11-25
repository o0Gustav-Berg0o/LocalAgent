# Module 1: Setup

**Goal:** the app starts on a machine you have not seen before. It scans the machine by itself, shows what it found for you to verify or change, asks the few questions the machine cannot answer, suggests a model, and guides the installation. Nothing is downloaded without your yes. Each start asks which folder to work in.

**Why a module of its own:** everything after this assumes the app knows its own settings and its folders. This module makes that true before any agent behaviour is built.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 01 | [First-run setup](01-first-run-setup/README.md) | about 3 hours | a read-only scan that runs by itself, a verify-or-change screen, a main folder, a working-folder choice at each start, a model suggestion, a guided install, and a guarded download |

## Before you start

- Module 0 read (the RAG course).

## What you have at the end

- A `/setup` command that can be re-run safely.
- A config file the app reads on every start, with no secrets in it.
- A scan that shows the source of each value, and a verify step where the user can change any of them.
- A working-folder question at every start, kept apart from the main folder.
- A download step that never writes until the user has seen the source, the name, the size, and the destination.
