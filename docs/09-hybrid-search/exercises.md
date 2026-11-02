# Lesson 9 — Exercises

## Exercise 1 — Prove the failure mode first

**Goal:** before building anything, confirm the problem you're solving actually exists in your data.

**Do**
1. Add 2–3 notes containing specific identifiers or error codes (e.g. a fake error constant like `WIFI_EVENT_STA_DISCONNECTED`, or a product code).
2. Run `/search` for that exact identifier.
3. Also add a few questions using these identifiers to your Lesson 8 eval set, with the correct expected source.

**Observe**
- Does vector search rank the chunk containing the exact identifier where you'd expect, or does a topically-similar-but-wrong chunk outrank it?

## Exercise 2 — Implement BM25

**Goal:** a second, independent ranking signal.

**Do**
1. Implement (or integrate) BM25 scoring over your existing chunk text.
2. Add a `/search-bm25 <query>` command to inspect its rankings in isolation, the same way `/search` shows vector rankings.
3. Run the identifier queries from Exercise 1 through `/search-bm25` and compare.

## Exercise 3 — Merge with RRF

**Goal:** combine both signals into one ranking.

**Do**
1. Implement RRF: for each chunk, sum `1 / (k + rank)` across the vector ranking and the BM25 ranking it appears in (use `k = 60` as a starting point).
2. Replace (or offer as an option alongside) your existing search with this hybrid ranking.
3. Re-run your full Lesson 8 eval harness: once with vector-only, once with hybrid.

**Observe**
- What's the recall@k delta overall, and specifically on the identifier-based questions you added in Exercise 1?
- Did hybrid search hurt any of your paraphrase-style questions from earlier lessons, or was it a clean win?

**Discuss**
- Why does RRF use rank position rather than directly combining the raw BM25 and cosine-similarity scores?
- What value of `k` in the RRF formula would favor BM25-style exact matches more? What would favor broad semantic recall more?
- Where would a third signal (e.g. a reranker model) fit into this pipeline, and what would it cost?
