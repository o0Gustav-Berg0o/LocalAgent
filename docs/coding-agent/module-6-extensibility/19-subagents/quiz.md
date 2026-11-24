# Module 6, Lesson 19 — Quiz

Write your own answer to each question first, then check the answer key.

1. What are the four differences between a subagent and the main loop?
2. Why does a subagent's tool list need to be enforced in code, not in its prompt?
3. A subagent runs out of budget. What should its result say, and what should the main agent tell the user?
4. Why don't permissions the user gave the main agent carry over to a subagent?
5. Why must a subagent not be able to start another subagent?
6. A subagent's answer is wrong and the main agent accepts it. What should have caught it?

---

## Answer key

1. Its own history, its own tool list, its own budget, and a fixed output. It runs through the same loop, not a different one.
2. A sentence in a prompt is a request the model may ignore. The list is checked before each call, so a tool outside it cannot run, whatever the model writes.
3. The result should be marked incomplete, with the answer so far and a note that the task was not finished. The main agent should show that status and not present the answer as final.
4. The user approved a request in the context they understood. A subagent may ask for something broader, and each of its actions must be checked on its own.
5. Recursion can spend the whole budget without limit, and the tree of work becomes impossible to follow or stop.
6. The evidence: file names and line numbers that the main agent could check against the files. An answer without evidence cannot be checked.

---

## Scoring

- 5 or 6 correct: ready for Lesson 20.
- 3 or 4 correct: re-read sections 3.3, 3.4, and 3.5 of the README, then repeat Exercises 3 and 5.
- Fewer than 3: repeat Exercises 1 to 4.
