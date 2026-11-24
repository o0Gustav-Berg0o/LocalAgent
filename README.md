# LocalAgent Course — Build a Local RAG Agent From Scratch

A self-paced, hands-on course for building a local, tool-using RAG (Retrieval-Augmented Generation) agent in C#/.NET, running entirely against your own GPU via [llama.cpp](https://github.com/ggml-org/llama.cpp). No cloud APIs, no API keys, full visibility into every step the agent takes.

This repository contains **guides, slides, exercises, and review checklists only** — no starter code. You write the application yourself, lesson by lesson. Each lesson teaches one concept deeply, then has you implement or modify a small, working piece of it.

## Why build an agent instead of using a framework?

Frameworks like LangChain or Semantic Kernel hide the agent loop behind abstractions. That's great for production, bad for learning. This course has you hand-write the loop — `LLM → tool call → result → LLM` — so you see exactly what the model is sent, what it asks for, and what comes back. Once you've built it by hand, framework abstractions stop being magic.

## Prerequisites

- Comfortable with C# and basic async/await.
- A machine with an NVIDIA GPU (~8 GB VRAM) *or* patience to run on CPU.
- Willingness to read model output critically — small local models (3–4B parameters) make mistakes, and debugging those mistakes is where most of the learning happens.

## How this course is structured

Each lesson lives in `docs/NN-lesson-slug/` and has three parts:

| File | Purpose |
|---|---|
| `README.md` | The lesson itself: concept explanation, why it matters, what you'll build. Written to be read before or alongside the linked slide deck. |
| `exercises.md` | Hands-on tasks in **Goal / Do / Observe / Discuss** format. This is where you actually write code and run the agent. |
| `review-checklist.md` | A self-check: expected behaviors, example trace output, and questions to confirm you actually understood the lesson, not just completed it. |

Each lesson's `README.md` also links to a **Google Slides deck** covering the same material visually — useful for presenting to others or for a quick recap.

## Lesson map

The lessons are grouped into modules. Each module is a folder under `docs/course-1-local-rag/`, with its own overview. Lesson numbers are unchanged.

| Module | Lessons |
|---|---|
| [Module 1: Setup and the loop](docs/course-1-local-rag/module-1-setup-and-loop/README.md) | 0 [Setup & Orientation](docs/course-1-local-rag/module-1-setup-and-loop/00-setup/README.md), 1 [Agents, Tools & the Loop](docs/course-1-local-rag/module-1-setup-and-loop/01-agents-tools-loop/README.md) |
| [Module 2: Retrieval](docs/course-1-local-rag/module-2-retrieval/README.md) | 2 [Embeddings & Semantic Search](docs/course-1-local-rag/module-2-retrieval/02-embeddings-semantic-search/README.md), 3 [Chunking Strategies](docs/course-1-local-rag/module-2-retrieval/03-chunking-strategies/README.md), 4 [Retrieval, Grounding & Hallucination](docs/course-1-local-rag/module-2-retrieval/04-retrieval-grounding-hallucination/README.md), 9 [Beyond Vectors: Hybrid Search](docs/course-1-local-rag/module-2-retrieval/09-hybrid-search/README.md) |
| [Module 3: Agent behaviour and safety](docs/course-1-local-rag/module-3-agent-behaviour/README.md) | 5 [Tool Design for Small Models](docs/course-1-local-rag/module-3-agent-behaviour/05-tool-design-small-models/README.md), 6 [Context Window & Memory](docs/course-1-local-rag/module-3-agent-behaviour/06-context-window-memory/README.md), 7 [Prompt Injection & the Confirmed Write Tool](docs/course-1-local-rag/module-3-agent-behaviour/07-prompt-injection-confirmed-write/README.md) |
| [Module 4: Measuring and architecture](docs/course-1-local-rag/module-4-measuring-and-architecture/README.md) | 8 [Measuring Quality: an Eval Harness](docs/course-1-local-rag/module-4-measuring-and-architecture/08-eval-harness/README.md), 10 [Separating the Core: a Shared Library](docs/course-1-local-rag/module-4-measuring-and-architecture/10-shared-library/README.md), 11 [Exposing Tools via MCP](docs/course-1-local-rag/module-4-measuring-and-architecture/11-mcp-exposure/README.md) |
| [Module 5: Capstone](docs/course-1-local-rag/module-5-capstone/README.md) | 12 [Capstone & Retrospective](docs/course-1-local-rag/module-5-capstone/12-capstone/README.md) |

## What you'll have built by the end

A console agent that:
- Answers questions grounded in your own markdown notes, with citations.
- Refuses to answer when the notes don't contain the answer.
- Combines keyword and semantic search for better recall.
- Can write new notes, but only with your explicit confirmation.
- Has a small automated test suite measuring retrieval quality.
- Exposes its tools over MCP so other agents (including Claude) can use them.

## Credits

Architecture and lab format inspired by a local teaching project built around Qwen3 + llama.cpp. This course restructures and extends those ideas into a self-paced curriculum.
