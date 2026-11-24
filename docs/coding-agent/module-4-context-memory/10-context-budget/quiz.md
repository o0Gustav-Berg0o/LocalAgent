# Module 4, Lesson 10 — Quiz

Write your own answer to each question first, then check the answer key.

1. Name the five parts of a request that use the context window.
2. The context window is 8192 tokens and the reserve is 1024. What is the budget, and why is it not 8192?
3. Why should tool results be counted once, inside the history, and not twice?
4. When would a calibration factor from one model be wrong for another?
5. Why should a threshold message print once when the level changes, and not on every turn?
6. Why does the app refuse a request before sending it, instead of letting the server cut it?

---

## Answer key

1. The system prompt, the tool descriptions, the memory, the history, and the tool results (which are part of the history).
2. The budget is 8192 minus 1024, which is 7168 tokens. The reserve must be excluded, because the model needs room to write its answer, and a full window leaves none.
3. Counting a tool result twice inflates the budget by the size of every file read, so the warnings fire too early and the numbers stop matching the request.
4. Each model has its own tokenizer and its own way of splitting text. A factor measured on one model can be wrong for another, and a factor measured on code can be wrong for Swedish prose.
5. A message on every turn becomes noise, and the user learns to ignore it. Printing on change tells the user something new happened.
6. A request the server cannot accept fails or is cut silently, and the model answers from incomplete input without knowing it. Refusing before sending makes the limit visible and gives the user the choice.

---

## Scoring

- 5 or 6 correct: ready for Lesson 11.
- 3 or 4 correct: re-read sections 3.1 and 3.2 of the README, then repeat Exercises 1 and 4.
- Fewer than 3: repeat Exercises 1 to 6.
