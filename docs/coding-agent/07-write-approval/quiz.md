# Module 3, Lesson 7 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why are propose, review, and apply three separate functions?
2. A user approves a change, then edits the file in an editor, then the change is applied. What should happen, and which step prevents the wrong result?
3. Why is the fingerprint computed from the bytes, not from the text?
4. Why must the temporary file be written in the same folder as the target?
5. A diff shows the whole file as changed, though only the line endings differ. What is wrong, and how do you fix it?
6. Why should the tool refuse to create a folder that does not exist?

---

## Answer key

1. Each step can be tested alone. A proposal can be checked without writing, and an apply can be tested with a fake approval.
2. The apply must be refused. The fingerprint check, run immediately before the write, compares the file on disk with the version the proposal was computed against, and refuses on a mismatch.
3. A text read can change the bytes through encoding handling. The fingerprint must match the bytes on disk, or the comparison can be wrong.
4. The move that makes the write atomic only works within one volume. A temporary file in another folder may be on another drive, where the move is a copy and is not atomic.
5. The comparison is treating the line ending as part of the text. Split into lines, and remove a trailing carriage return before comparing.
6. A missing folder usually means a typo in the path. Creating it silently puts the change in the wrong place and hides the mistake.

---

## Scoring

- 5 or 6 correct: ready for Lesson 8.
- 3 or 4 correct: re-read sections 3.3 and 3.4 of the README, then repeat Exercises 3 and 4.
- Fewer than 3: repeat Exercises 1 to 5.
