# Module 4, Lesson 12 — Quiz

Write your own answer to each question first, then check the answer key.

1. Name the three kinds of memory, and say which of them survive a session.
2. Why does an instruction file cost tokens on every request, and what does that imply for its length?
3. Why must an agent note be approved before it is saved?
4. The user's instruction file says "use tabs", and an agent note says "use spaces". What should the model do, and who decides?
5. Why must memory never contain a key, and what does the secret check miss?
6. Why should the git check in `/memory` never stage or commit a file?

---

## Answer key

1. The user's project instructions and the agent's approved notes survive a session. The conversation does not, unless it was saved as a session (Lesson 13) or written into a note.
2. The file is part of the system prompt, which is sent with every request, so its size is paid on each turn. It should be short enough that the budget stays comfortable.
3. A note is trusted in every future session. A note written because a document asked for it would become an injection that survives restarts. Approval keeps the user in control of what the model believes.
4. The model should not choose silently. The conflict should be shown to the user, who decides which rule stands.
5. A memory file is saved and often committed, and the model reads it on every request. A key in memory is a copy of the secret in a place the user did not expect. The secret check misses formats it does not know, so it is a heuristic and not a guarantee.
6. A check that stages or commits a file is a write, and needs the same approval as any write (Lesson 7). The view must only read.

---

## Scoring

- 5 or 6 correct: ready for Lesson 13.
- 3 or 4 correct: re-read sections 3.1, 3.4, and 3.6 of the README, then repeat Exercises 3 and 4.
- Fewer than 3: repeat Exercises 1 to 4.
