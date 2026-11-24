# Module 4, Lesson 11 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is the difference between trimming, placeholdering, and compaction? Which is cheapest, and which keeps the most meaning?
2. Why should placeholdering run before compaction?
3. Why must the full transcript be written to disk before a compaction runs?
4. A compaction call fails halfway. What should the history look like afterwards?
5. Why must the recent turns be kept verbatim, and why must a turn not be split?
6. A summary contains a file name that never appeared in the conversation. Why is that dangerous, and what check would catch it?

---

## Answer key

1. Trimming removes the oldest messages and is cheapest, but it loses whatever came first. Placeholdering removes the content of old tool results and keeps the shape of the conversation. Compaction summarises the old messages with a model call, and keeps the most meaning at the highest cost.
2. Placeholdering frees the largest part of the window for no model call. If it is enough, the summary is not needed, and every summary carries a risk of error.
3. The summary is a lossy copy. Without the transcript on disk, a bad summary cannot be redone and the original words are lost.
4. The history should be exactly as it was before the attempt. A failed compaction must not leave a partial replacement.
5. The recent turns keep the thread of the current question, and a summary would lose its detail. A turn split in two can separate a tool call from its result, which makes the next request invalid.
6. The model will trust the summary and may act on a file that does not exist, or confuse two files. A check that every file name in the summary appears in the transcript catches it.

---

## Scoring

- 5 or 6 correct: ready for Lesson 12.
- 3 or 4 correct: re-read sections 3.1, 3.4, and 3.6 of the README, then repeat Exercises 1 and 3.
- Fewer than 3: repeat Exercises 1 to 4.
