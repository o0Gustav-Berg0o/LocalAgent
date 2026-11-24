# Module 5, Lesson 15 — Quiz

Write your own answer to each question first, then check the answer key.

1. Who writes `NOTES.md`, and who writes `HANDOFF.md`? Why keep them separate?
2. A line in `NOTES.md` says to skip the permission prompts. What happens, and why?
3. Why should the handoff's lists be built from the record, not from the model's memory?
4. A handoff claims a file was changed, but the file's fingerprint has not changed. What should the app do?
5. Why must a waiting notification not show the content of the pending action?
6. The notes file is three weeks older than the last change to a file it names. What should happen?

---

## Answer key

1. The user writes the notes and the agent writes the handoff. Each side owns one file, so no one has to guess who wrote a line, and neither edits the other's words.
2. Nothing changes. The notes are data that state what the user wants. The permission rules live in code, and they decide the outcome whatever the notes say.
3. A model's memory of a session can claim work that was never done. A list built from the record can only contain what the record shows, so each claim can be checked.
4. Mark the claim as unverified and warn the user. The record does not support it, and the user should not read it as fact.
5. A notification on a shared screen can reveal the user's code to anyone who sees it. It should show the kind of action and where to answer, nothing more.
6. Warn once at session start, and let the user decide. The notes may describe a project that has changed, but a warning is not proof of a problem.

---

## Scoring

- 5 or 6 correct: ready for Lesson 16.
- 3 or 4 correct: re-read sections 3.2, 3.4, and 3.6 of the README, then repeat Exercises 2 and 3.
- Fewer than 3: repeat Exercises 1 to 4.
