# Lesson 9 — Beyond Vectors: Hybrid Search

**Slides:** _link added once the deck is created_

## Learning objectives

- Understand a concrete class of queries where pure vector search underperforms.
- Implement BM25 keyword scoring alongside your existing vector search.
- Merge both rankings with Reciprocal Rank Fusion (RRF) and measure the result with your Lesson 8 harness.

## Where vector search falls short

Vector search is excellent at matching *meaning*. It is comparatively weak at matching *exact tokens that carry little semantic content on their own* — error codes, identifiers, API names, version numbers. `WIFI_EVENT_STA_DISCONNECTED` doesn't "mean" anything to an embedding model the way "the connection was lost" does; the model has no strong semantic signal to anchor on, so an exact-identifier query can rank a chunk containing that literal string lower than you'd expect, especially if other chunks happen to be topically similar.

**Keyword search (BM25)** scores documents by term frequency, weighted by how rare/informative each term is across the whole collection. It's the opposite tool for the opposite failure mode: it excels at exact and near-exact term matches, and has no concept of meaning at all.

Neither approach is strictly better — they fail on different things. Combining both is called **hybrid search**.

## Reciprocal Rank Fusion (RRF)

You now have two separate rankings of the same chunks (one from BM25, one from vector similarity), on two different, incomparable scales. RRF sidesteps the scale problem by using each chunk's *rank* (1st, 2nd, 3rd...) rather than its raw score:

```text
RRF_score(chunk) = sum over each ranking list containing the chunk of: 1 / (k + rank_in_that_list)
```

(`k` is a small constant, often 60, that dampens the impact of rank 1 vs rank 2 differences.) Sum a chunk's contributions across both lists, sort by total RRF score — chunks that rank well in *both* lists rise to the top, and a chunk that only one method found still gets a chance.

## What you'll build

- A BM25 scorer over your existing chunks (term frequency, inverse document frequency, length normalization — or use a small existing library if you prefer to focus on integration over reimplementation).
- An RRF merge step combining BM25 and vector rankings into one list.
- A measured comparison, using your Lesson 8 harness, of recall@k for vector-only vs. hybrid search — specifically on queries containing exact identifiers.
