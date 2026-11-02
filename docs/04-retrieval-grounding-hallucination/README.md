# Lesson 4 — Retrieval as a Tool, Grounding & Hallucination

**Slides:** _link added once the deck is created_

## Learning objectives

- Wire search into the agent loop as a tool the model *chooses* to call.
- Get asymmetric query embedding right, and see what happens when it's wrong.
- Distinguish a model citing retrieved text from a model confidently making things up.

## Retrieval as a tool, not as auto-injection

The naive way to build RAG is: on every user message, always run search and paste the top chunks into the prompt before calling the model. That's simple but wasteful — a greeting or a pure arithmetic question gets search results jammed into context for no reason, burning tokens and adding noise.

Instead, expose search as a tool (`search_notes(query: string)`) and let the model decide *if* and *with what query* to call it. This costs nothing on turns that don't need retrieval, and lets the model reformulate or search again if the first attempt misses. The tradeoff: a small model might skip searching when it should, or search with a poor query — which is why the system prompt should explicitly instruct it to search first for any question about the notes.

## Getting asymmetric search right

Recall from Lesson 2: some embedding models expect queries and documents to be embedded differently. Documents get embedded as-is; queries get a task instruction prefix (e.g. `"Instruct: Given a question, retrieve relevant passages\nQuery: <question>"`). This is called **asymmetric search**. The failure mode is silent — nothing errors, search just quietly gets worse. This is one of the "quiet config bugs" that plague real RAG systems, and worth deliberately breaking once so you recognize it later.

## Grounding vs. hallucination

**Grounding** means the model's answer is actually built from the retrieved text, and cites where it came from (`file.md > heading`). **Hallucination** is the model answering from its own training data — plausible-sounding, but not actually backed by anything you gave it, and impossible to distinguish from a grounded answer just by reading the prose.

The system prompt is your main lever here: explicitly instruct the model to base answers only on tool results, and to say when the notes don't contain an answer. Verify it actually does — a `MinScore` cutoff that makes `search_notes` return "no relevant passages found" when nothing in your notes is relevant is what makes "I don't know" the correct behavior to begin with. Without that cutoff, the model might get served four marginally-related chunks and have no outs.

## What you'll build

- `search_notes` as a real tool in your agent loop, with a system-prompt instruction to use it first for any question about your notes.
- Correct asymmetric query embedding, with a deliberate experiment removing it to see the effect.
- A `MinScore` threshold and a "no relevant passages found" path, plus a system prompt that makes the model say "I don't know" instead of filling the gap from its own training data.
