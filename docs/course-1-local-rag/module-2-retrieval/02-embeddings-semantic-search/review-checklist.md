# Lesson 2 — Review Checklist

- [ ] `/search` returns results with scores, with no chat model call involved — confirm by watching network traffic or server logs: only the embedding server should see traffic.
- [ ] A paraphrased query finds the same note as the exact wording.
- [ ] An unrelated query produces visibly lower scores than a genuine match.
- [ ] Re-running the app without changing any notes does not re-embed anything (check via timing or trace output).
- [ ] Changing one note's content re-embeds only that note, not the whole set.
- [ ] Your index persists to disk and reloads correctly after restarting the app.

## Self-check questions

1. In your own words, what does a cosine similarity of exactly 1.0 mean? Of 0.0?
2. Why is a vector-only approach likely to struggle with an exact identifier or error code lookup?
3. What would happen to your stored vectors if you later embedded everything with a *different* embedding model but forgot to rebuild the index?
