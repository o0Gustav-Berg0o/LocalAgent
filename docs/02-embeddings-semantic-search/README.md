# Lesson 2 — Embeddings & Semantic Search

**Slides:** _link added once the deck is created_

## Learning objectives

- Understand what an embedding actually is and why similar meaning → similar vector.
- Implement a minimal vector store and cosine-similarity search, with no LLM involved.
- See search work on paraphrases, and fail on topics that genuinely aren't there.

## What is an embedding, really?

An embedding model maps a piece of text to a fixed-length list of numbers (e.g. 1024 floats). That list is a point in a high-dimensional space. The model is trained so that texts with similar *meaning* end up as nearby points, regardless of exact wording.

This is why semantic search can match *"reconnect after losing network"* to a note titled *"WiFi reconnection handling"* even though they don't share a single word. Keyword search cannot do that; vector search is built for exactly this.

**Cosine similarity** measures how close two vectors point in the same direction (ignoring magnitude) — a number from -1 to 1, usually 0 to 1 in practice for embeddings. Higher means "more similar in meaning."

## Why build retrieval before touching the LLM?

Because retrieval quality is a completely separate problem from the agent loop, and debugging it is far easier without an LLM in the way. If your search returns the wrong chunk, no amount of prompt engineering on the chat model will fix that — you have to fix retrieval itself. This lesson has you build and test search as a *standalone, inspectable* piece, with a `/search` command that queries it directly and prints scores — no model call involved.

## Asymmetric search, previewed

You'll notice something odd later: embedding a *question* and embedding a *document* aren't quite symmetric operations for some models. Some embedding models (including the Qwen3-Embedding family) are trained to expect an instruction prefix on the *query* side only (e.g. `"Instruct: Given a question, retrieve relevant passages\nQuery: <question>"`), while documents are embedded plain. Get this wrong and search quietly gets worse — no error, just lower quality. This lesson introduces the mechanism; Lesson 4 has you break it on purpose to feel the effect.

## What you'll build

- A minimal in-memory vector store: a list of `(text, vector)` pairs, with a function that embeds a query and returns the top-K most similar entries by cosine similarity.
- Persistence: save the store to a JSON file so you don't re-embed everything on every run.
- A `/search <text>` console command that runs retrieval alone and prints each hit's score and source text — your primary debugging tool for the rest of the course.
