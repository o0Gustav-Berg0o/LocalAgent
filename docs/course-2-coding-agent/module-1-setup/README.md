# Module 1: Setup

**Goal:** the app starts on a machine you have not seen before, asks what it needs to know, looks at the hardware, suggests a model, and guides the installation. Nothing is downloaded without your yes.

**Why a module of its own:** everything after this assumes the app knows its own settings. This module makes that true before any agent behaviour is built.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 01 | [First-run setup](01-first-run-setup/README.md) | about 3 hours | a setup wizard, a read-only hardware scan, a model suggestion, a guided install, and a guarded download |

## Before you start

- Module 0 read (the RAG course).

## What you have at the end

- A `/setup` command that can be re-run safely.
- A config file the app reads on every start, with no secrets in it.
- A download step that never writes until the user has seen the source, the name, the size, and the destination.
