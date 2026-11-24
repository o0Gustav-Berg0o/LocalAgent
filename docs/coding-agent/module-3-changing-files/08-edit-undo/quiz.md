# Module 3, Lesson 8 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why is exact replacement safer than rewriting a whole file?
2. An old text occurs in two places in a file. What should the tool do, and what should the refusal say?
3. Two edits to one file, and the second one fails. What should happen to the first?
4. Why must the snapshot be taken before the write, not after?
5. The user changed a file by hand after an agent change. What should undo do, and why?
6. Why must undo restore all files of a change, or none?

---

## Answer key

1. A rewrite reproduces every line, and each reproduced line can change by mistake without review. An edit names only the text that changes, so the rest of the file cannot change unnoticed.
2. The tool should refuse, because it cannot know which occurrence was meant. The refusal should list the line numbers and ask for more surrounding text so the match is unique.
3. Nothing should be written. The set of edits is one change, so a failure in one part refuses the whole set, and the refusal names the failing edit.
4. A snapshot after the write holds the new content, so undo would restore nothing useful. The snapshot must hold the content the file had before the change.
5. Undo should refuse and list the file. The fingerprint after the agent's change no longer matches, so restoring would discard the user's manual work.
6. A partial undo leaves the project in a state that matches neither the before nor the after. The project is inconsistent, and the user may not notice until a build fails.

---

## Scoring

- 5 or 6 correct: ready for Lesson 9.
- 3 or 4 correct: re-read sections 3.1 and 3.5 of the README, then repeat Exercises 1 and 5.
- Fewer than 3: repeat Exercises 1 to 5.
