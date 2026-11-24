# Lesson 9 — Review Checklist

- [ ] You demonstrated a concrete case (an identifier/error-code query) where vector-only search underperformed.
- [ ] You have a working, independent BM25 ranking you can inspect on its own via `/search-bm25`.
- [ ] You implemented RRF to merge BM25 and vector rankings using rank position, not raw scores.
- [ ] You ran your Lesson 8 harness on both vector-only and hybrid search and can state the actual recall@k numbers for each.
- [ ] You checked that hybrid search didn't regress the paraphrase-style questions that vector search was already good at.

## Self-check questions

1. Why can't you just average or add raw BM25 and cosine similarity scores directly?
2. Give one example query type where BM25 alone would badly fail, matching the structure of Lesson 2's "vector-only" discussion.
3. If your eval harness showed hybrid search was *worse* than vector-only overall, what would you check before concluding hybrid search is a bad idea?
