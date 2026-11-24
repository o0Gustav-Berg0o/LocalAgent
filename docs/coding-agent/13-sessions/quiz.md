# Module 4, Lesson 13 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is the difference between a session, the working history, and project memory?
2. Why is a session file append-only, and what does a crash damage?
3. Why must a resume never run a tool again?
4. A session file has a line that is not valid JSON. What should the reader do?
5. Why should the picker sort by the time of the last event, not the file's modification time?
6. A session holds a tool result whose call is missing. What should a resume do?

---

## Answer key

1. The session is the saved record of a conversation. The working history is the view of it that is sent to the model now, and it may be shorter after compaction. Project memory holds facts about the project and is separate from both.
2. Appending means earlier lines are never rewritten. A crash can damage only the last line, which the reader recognises and skips.
3. A tool may write, run a command, or change something outside the app. Repeating it during a resume would repeat an action the user approved once, without a new decision.
4. Skip the line, count it, and report its number. The rest of the session should still be usable.
5. Backups and copies can change the modification time, and a clock can be wrong. The time of the last event is in the record itself.
6. The resume should add a result that says it was not recorded, and tell the user, so the history stays valid for the model.

---

## Scoring

- 5 or 6 correct: ready for Lesson 14.
- 3 or 4 correct: re-read sections 3.1, 3.2, and 3.5 of the README, then repeat Exercises 2 and 4.
- Fewer than 3: repeat Exercises 1 to 4.
