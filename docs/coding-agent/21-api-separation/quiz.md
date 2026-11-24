# Module 7, Lesson 21 — Quiz

Write your own answer to each question first, then check the answer key.

1. What belongs in the core, and what belongs in the frontend? Give one example of each.
2. Why does the core emit events instead of writing to the screen itself?
3. Why does an approval need an identifier, and what happens to an answer with the wrong one?
4. Why can a test that asserts on events be more reliable than a test that reads the console output?
5. Renaming an event field is a breaking change, but adding an optional field is not. Why?
6. A frontend sends a command the core does not expect. What should the core do?

---

## Answer key

1. The core holds the loop, tools, permissions, the budget, and sessions. The frontend holds the screen, the keyboard, the status line, and the layout. Example for the core: the permission check. Example for the frontend: the colour of a diff.
2. So that the core does not depend on any particular screen, and any frontend can show the same events in its own way.
3. An approval answers one specific request. The identifier ties the answer to that request, so an answer for an old or unknown request cannot approve something the core did not ask about.
4. Console output depends on wording, colour, and layout, and changes with any screen change. Events carry the meaning, and a test that asserts on them checks the behaviour that matters.
5. An old frontend that does not know a new optional field can ignore it and keep working. A renamed field has no value for the old frontend to read, so it breaks.
6. The core should refuse it with a message, and the refusal should be an event the frontend can show. It should not guess what the command meant.

---

## Scoring

- 5 or 6 correct: ready for Lesson 22.
- 3 or 4 correct: re-read sections 3.1, 3.4, and 3.6 of the README, then repeat Exercises 2 and 3.
- Fewer than 3: repeat Exercises 1 to 4.
