# Module 2, Lesson 4 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is the difference between a setting, session state, and a mode? Give one example of each.
2. Why is effort defined in code with a fixed table, rather than as a free number in the config?
3. A user types `8192` for the context size, but the server was started with 4096. What should happen, and where should the check live?
4. Why must the atomic save write a temporary file before it moves the new file into place?
5. A user presses Esc with a pending change. What are the three possible answers, and what happens to the file in each case?
6. Why must the menu still show a marker when `NO_COLOR` is set?

---

## Answer key

1. A setting persists in the file, for example the model name. Session state exists during a run or in a saved session, for example the number of turns. A mode is a per-session choice, for example code or cowork.
2. The effort levels change what the user gets, so their meaning must be consistent across the app and testable. A free number lets a user set combinations the code was never tested with.
3. The menu refuses the value and shows the server's value. The check belongs in the validator, which runs before the save, and it compares against the server value that the app knows.
4. Writing the temporary file first means the real file is never deleted before the new one is complete. A crash during the write leaves the old file intact.
5. Save writes exactly the pending values. Discard leaves the file unchanged and reverts the screen. Keep editing returns to the row with the pending value still shown and nothing written.
6. Colour is only one way to show the selection. Without the marker, a user with no colour cannot see which row is selected.

---

## Scoring

- 5 or 6 correct: ready for Lesson 5.
- 3 or 4 correct: re-read sections 3.1 and 3.4 of the README, then repeat Exercise 3.
- Fewer than 3: repeat Exercises 1 and 2.
