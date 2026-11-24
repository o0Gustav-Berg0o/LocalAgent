# Module 0: Foundations (Course 1)

**Goal:** the pieces of Course 1 that Course 2 builds on. Read these before Module 1. You do not repeat them in Course 2; you use them.

**Time:** the Course 1 lessons, as you planned them (about 4 hours each).

## Lessons to read first

| Course 1 lesson | What Course 2 uses it for |
|---|---|
| [1. Agents, tools and the loop](../../01-agents-tools-loop/README.md) | the loop that every later lesson extends |
| [2. Embeddings and semantic search](../../02-embeddings-semantic-search/README.md) | vectors and search without a model |
| [3. Chunking strategies](../../03-chunking-strategies/README.md) | splitting files, used by the indexer and `/chunks` |
| [4. Retrieval, grounding and hallucination](../../04-retrieval-grounding-hallucination/README.md) | search as a tool, and saying "I don't know" |
| [5. Tool design for small models](../../05-tool-design-small-models/README.md) | descriptions and names for every tool |
| [6. Context window and memory](../../06-context-window-memory/README.md) | basic trimming, the start of Module 4 |
| [7. Prompt injection and confirmed writes](../../07-prompt-injection-confirmed-write/README.md) | the approval pattern behind Modules 2 and 3 |
| [8. Eval harness](../../08-eval-harness/README.md) | the measurement used for every change |
| [9. Hybrid search](../../09-hybrid-search/README.md) | keyword and vector ranking |

## Before you start Module 1

- You can run `/search`, `/eval`, and the agent loop in the Course 1 app.
- You know what a confirmed write is, and why approval is enforced in code.
- Both `llama-server` processes start on your machine.
