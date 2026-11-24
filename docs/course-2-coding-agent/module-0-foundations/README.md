# Module 0: Foundations (the RAG course)

**The RAG course** (Course 1 in this repository) builds **LocalAgent**, a notes assistant that searches your own markdown files, answers from them, and measures its own search with an eval harness. It lives in `docs/course-1-local-rag/`.

**Course 2** (this course) builds a coding agent. It reuses ideas and parts from LocalAgent, so Module 0 is a reading list, not a set of new lessons.

**Goal:** read the RAG course lessons below before Module 1. Course 2 does not repeat them; it refers back to them and uses what they taught.

**Time:** each lesson of the RAG course takes the time its own page gives.

## Lessons to read first

| the RAG course lesson | What Course 2 uses it for |
|---|---|
| [1. Agents, tools and the loop](../../course-1-local-rag/module-1-setup-and-loop/01-agents-tools-loop/README.md) | the loop that every later lesson extends |
| [2. Embeddings and semantic search](../../course-1-local-rag/module-2-retrieval/02-embeddings-semantic-search/README.md) | vectors and search without a model |
| [3. Chunking strategies](../../course-1-local-rag/module-2-retrieval/03-chunking-strategies/README.md) | splitting files, used by the indexer and `/chunks` |
| [4. Retrieval, grounding and hallucination](../../course-1-local-rag/module-2-retrieval/04-retrieval-grounding-hallucination/README.md) | search as a tool, and saying "I don't know" |
| [5. Tool design for small models](../../course-1-local-rag/module-3-agent-behaviour/05-tool-design-small-models/README.md) | descriptions and names for every tool |
| [6. Context window and memory](../../course-1-local-rag/module-3-agent-behaviour/06-context-window-memory/README.md) | basic trimming, the start of Module 4 |
| [7. Prompt injection and confirmed writes](../../course-1-local-rag/module-3-agent-behaviour/07-prompt-injection-confirmed-write/README.md) | the approval pattern behind Modules 2 and 3 |
| [8. Eval harness](../../course-1-local-rag/module-4-measuring-and-architecture/08-eval-harness/README.md) | the measurement used for every change |
| [9. Hybrid search](../../course-1-local-rag/module-2-retrieval/09-hybrid-search/README.md) | keyword and vector ranking |

## Before you start Module 1

- You can run `/search`, `/eval`, and the agent loop in the RAG course app.
- You know what a confirmed write is, and why approval is enforced in code.
- Both `llama-server` processes start on your machine.
