# Lesson 2 — Exercises

## Exercise 1 — Minimal vector store

**Goal:** a working, inspectable search index with zero LLM involvement.

**Do**
1. Create 5–10 short markdown notes about a topic you know well (so you can judge relevance yourself).
2. Write a function that embeds a piece of text via the embedding server and returns the float vector.
3. Embed every note as one chunk each (don't worry about splitting yet — that's Lesson 3) and store `(sourceFile, text, vector)` tuples in memory.
4. Implement cosine similarity and a `TopK(query, k)` function using it.
5. Serialize the store to a JSON file; load it back on startup instead of re-embedding unchanged files (hash each file's content to detect changes).

## Exercise 2 — `/search` as a standalone command

**Goal:** a tool for looking at retrieval quality directly.

**Do**
1. Add a `/search <text>` command to your console app that calls `TopK` and prints each result's score and a snippet of its text.
2. Run `/search` with the exact wording from one of your notes.
3. Run `/search` again with a paraphrase — different words, same meaning.
4. Run `/search` with something totally unrelated to any of your notes.

**Observe**
- Is there a clear score gap between a good match and the unrelated query? How big?
- Did the paraphrase still find the right note, despite no shared words?

**Discuss**
- What would a reasonable `MinScore` cutoff be, based on what you saw? What's the risk of setting it too high?
- When would plain keyword search (e.g. matching an exact error code like `WIFI_EVENT_STA_DISCONNECTED`) beat vector search outright?
