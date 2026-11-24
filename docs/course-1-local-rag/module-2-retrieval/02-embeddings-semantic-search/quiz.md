# Lesson 2 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. What property does an embedding model guarantee about texts with similar meaning?
2. What does cosine similarity actually measure?
3. Why build and test retrieval with a `/search` command before wiring it into the chat agent at all?
4. What is "asymmetric search," and which side of a query/document pair gets special treatment?
5. At this stage of the course (before Lesson 3), what does a `Chunk`'s `Vector` field actually represent?

## Answers

1. They end up as nearby points (similar vectors) in the embedding space, regardless of whether they share any of the same words.
2. How close two vectors point in the same direction, ignoring their length — a number typically between 0 and 1 for embeddings, where higher means more similar in meaning.
3. Because retrieval quality is a separate problem from the agent loop, and bugs in it are far easier to isolate and debug without a chat model in the way.
4. Some embedding models expect the query to be embedded with an instruction prefix (e.g. `"Instruct: ...\nQuery: ..."`), while documents are embedded plain, with no prefix — only the query side changes.
5. An embedding of the entire file's text — one vector per whole file. Splitting a file into smaller pieces doesn't happen until Lesson 3.
