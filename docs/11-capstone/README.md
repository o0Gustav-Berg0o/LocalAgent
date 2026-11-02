# Lesson 11 — Capstone & Retrospective

**Slides:** https://docs.google.com/presentation/d/19oA5M3CZJCXjnQPNkIMrejFCi96P8kZpvub0mZLScnQ/edit

## Learning objectives

- Consolidate everything into one coherent, demo-able agent.
- Review your own build critically, not just functionally.
- Decide, with evidence, what you'd change if you started over or extended this further.

## What "done" looks like

By now you have:

1. An agent loop you wrote by hand, with tool calling, trace output, and an iteration cap (Lesson 1).
2. Structure-aware chunking with heading-path context (Lesson 3).
3. Retrieval exposed as a tool the model chooses to call, with grounding and a working "I don't know" path (Lesson 4).
4. Tool descriptions tuned for a small model's limitations (Lesson 5).
5. Context-window-aware history management (Lesson 6).
6. A demonstrated prompt injection, and a write tool that only acts with explicit per-action confirmation (Lesson 7).
7. A labeled eval set and a recall@k harness you've actually used to make a decision (Lesson 8).
8. Hybrid search (BM25 + vector via RRF), measured against vector-only (Lesson 9).
9. The same tools exposed over MCP to an external client (Lesson 10).

This lesson has no new concept. It's where you step back and look at the whole thing as a system, not a sequence of exercises.

## What you'll do

- A full run-through demo: start from a clean index, ask a sequence of questions that exercises retrieval, grounding, a tool call, and (with confirmation) a note creation — in one sitting, narrating what's happening at each step as if teaching someone else.
- A written retrospective, using the review template below, covering what surprised you, what you'd change, and what's still weak.
- A short list of next steps if you kept extending this (reranking, a real vector database, OpenTelemetry, summarization-based history management — all mentioned as extension points throughout the earlier lessons but deliberately left undone here).
