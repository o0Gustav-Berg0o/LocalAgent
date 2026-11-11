# Lesson 9 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why does vector search tend to underperform on an exact identifier like `WIFI_EVENT_STA_DISCONNECTED`?
2. What does BM25 score, and what's its core weakness compared to vector search?
3. Why can't you just add BM25 scores and cosine-similarity scores together directly?
4. In the RRF formula, what does summing `1 / (k + rank)` across both lists accomplish for a chunk that ranks well in both?
5. What happens, under RRF, to a chunk that only one of the two methods found at all?

## Answers

1. An embedding model has no strong semantic signal to anchor such a token on — it doesn't "mean" anything the way a sentence does, so a chunk containing the literal string can rank lower than topically-similar chunks that don't actually contain it.
2. It scores by term frequency weighted by how rare or informative each term is across the collection — good at exact/near-exact matches, but it has no concept of meaning at all, so it can't match a paraphrase.
3. They're on two different, incomparable scales. RRF sidesteps this by using each chunk's rank position in each list instead of its raw score.
4. Its contributions from both lists add together, pushing it toward the top of the combined ranking — rewarding chunks that both methods agree are relevant.
5. It still gets a chance — it still contributes whatever score it earned from the one list it appeared in, rather than being excluded just because the other method missed it.
