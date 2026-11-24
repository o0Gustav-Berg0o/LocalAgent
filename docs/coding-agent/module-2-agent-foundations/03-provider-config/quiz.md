# Module 2, Lesson 3 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is a seam, and what must the loop not import for the seam to work?
2. Why does the context size exist in two places, and which one decides when the app compacts the history?
3. Give one value that belongs in the config file and one that belongs in code, and explain the difference.
4. A config file has `"contextSize": "8192"` (with quotes). What should the validator do, and why?
5. Why must the validator never overwrite the user's file on disk?
6. A request fails with "connection refused". Write the message the user should see, in one or two sentences.

---

## Answer key

1. A seam is a place where one part can change without touching the others. The loop must not import the server client library, or changing the server would require changing the loop.
2. The server has a context size fixed at its start, and the app has a setting that decides when to compact. If they disagree, the app sends requests the server cannot handle. The app's setting decides compaction, so the server's value must be shown and checked against it.
3. The port of the chat server is a config value: it differs between machines. A search threshold such as 0.3 is a tuning value of the code: changing it changes the meaning of the search, so it stays with the code and is tested.
4. A strict validator should reject it with a warning and use the default, or accept the string and convert it. Either is valid if the decision is written down. The important part is that the app starts and the user is told.
5. The file belongs to the user, and the validator's corrected values are for this run only. Overwriting would silently discard whatever the user wrote, including fields this version does not understand.
6. Something like: "Could not reach the chat server at http://localhost:8080/v1 (connection refused). Start llama-server, and check that the port matches the chatEndpoint setting."

---

## Scoring

- 5 or 6 correct: ready for Lesson 4.
- 3 or 4 correct: re-read sections 3.1 and 3.3 of the README, then repeat Exercise 5.
- Fewer than 3: repeat Exercises 1 and 4. The seam is what the rest of the course depends on.
