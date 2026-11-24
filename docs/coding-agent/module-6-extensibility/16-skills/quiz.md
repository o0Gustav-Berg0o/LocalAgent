# Module 6, Lesson 16 — Quiz

Write your own answer to each question first, then check the answer key.

1. What are the two parts of a skill, and which one is sent on every request?
2. Why is a description like "Testing tips" a weak description for a skill?
3. Why must `load_skill` refuse a name that is not in the index?
4. A skill body says the permission rules do not apply. What happens, and why?
5. Two skills have the same name, one in the user folder and one in the project. Which is loaded, and how does the user see it?
6. Why should the index be measured, not assumed to be small?

---

## Answer key

1. The index entry (name and description) is sent on every request. The body is loaded only when the model calls the tool.
2. It does not say when to use the skill, so the model cannot choose it correctly. A description should describe the task that triggers the skill.
3. The index is the single source of truth. A name outside it could point to any file, and the refusal should list the valid names so the model can correct itself.
4. Nothing changes. The body is text the model may follow, but the permission rules are code. A skill cannot change code, and any request it makes goes through the same check.
5. The project skill wins, because it is loaded later and overrides the user skill. The index shows "(project override)" next to the name, so the choice is visible.
6. An index with many skills and long descriptions can cost more than the instructions it was meant to save. Measuring is the only way to know.

---

## Scoring

- 5 or 6 correct: ready for Lesson 17.
- 3 or 4 correct: re-read sections 3.1, 3.2, and 3.4 of the README, then repeat Exercises 2 and 3.
- Fewer than 3: repeat Exercises 1 to 4.
