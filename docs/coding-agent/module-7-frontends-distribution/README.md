# Module 7: Frontends and Distribution

**Goal:** the core of the agent is separated from the console, so other frontends can use it. Downloads from the web follow the same consent rules as models. The app is packaged so another person can install it on a clean machine.

**Why these together:** a second frontend proves the separation, a fetch tool is the most open door in the agent, and packaging is where every hidden assumption is found.

## Lessons

| Lesson | Title | Time | What you build |
|---|---|---|---|
| 21 | [API separation: core and frontends](21-api-separation/README.md) | about 3 hours | typed events and commands, identifier-matched approvals, and a second frontend |
| 22 | [Downloading files safely](22-downloads/README.md) | about 3 hours | a fetch tool with a host allowlist, checked redirects, and a consent-gated save |
| 23 | [Packaging and first run](23-packaging/README.md) | about 3 hours | a self-contained single-file release, a leak search, a first-run check, and checksums |

## Before you start

- Module 6 complete (the external-tool and hook rules are reused here).
- Module 1 (the download consent screen is reused in Lesson 22).

## What you have at the end

- A core that runs with no console, driven by a test.
- A fetch tool that cannot reach an unapproved host, and cannot run what it saves.
- A release that a person on a clean machine can install from the guide alone.
