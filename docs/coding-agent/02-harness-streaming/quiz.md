# Module 2, Lesson 2 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why does streaming change how long the wait feels, even when the total time is the same?
2. Why must a half-finished assistant message never be added to the history?
3. What is the difference between what the screen shows and what the history stores? Give one example of a decoration that must stay out of the history.
4. Why must the app wait for the stream to end before running a tool call that arrived in pieces?
5. A user presses Esc during a long answer. What should the history contain afterwards, and why?
6. The working indicator is still running after an exception. Which part of the code is missing, and what will the user see?
7. Time to first token is 1.8 seconds, and total time is 1.2 seconds. What is wrong, and where would you look?

---

## Answer key

1. The first words appear while the rest is still being produced, so the user sees progress early. The wait is the same length, but it feels shorter because the user is not staring at nothing.
2. The next request sends the history to the model. A half-finished sentence in the history makes the model continue something that was never finished, and its answers become confused.
3. The screen may show a coloured prefix, line wrapping, or a summary of tool calls. The history stores only the plain text the model wrote. Example: a coloured `Agent>` prefix must not be stored, or the model will imitate it.
4. The arguments of a tool call arrive in fragments that are valid JSON only when complete. Running the tool with half its arguments produces errors that are hard to trace.
5. The history should contain the user's question and no partial answer. The question stays, so the model knows it was asked. Some designs add a short "cancelled" message, which is clearer to the model. Either choice is valid if it is written down.
6. A `finally` block that stops the indicator. Without it, the indicator keeps writing over the next prompt and the screen looks corrupted.
7. Total time cannot be shorter than time to first token, because the first token comes before the end. This is a timing bug: the end time is probably recorded before the stream finishes, or the two timestamps come from different calls. Check where each stopwatch starts and stops.

---

## Scoring

- 6 or 7 correct: ready for Lesson 3.
- 4 or 5 correct: re-read sections 3.3 to 3.5 of the README, and repeat Exercise 3.
- Fewer than 4: repeat Exercises 2 to 4. The history rule underlies everything after this lesson.
