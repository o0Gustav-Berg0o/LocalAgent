# Module 2, Lesson 6 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why must a permission check happen in code, not in the system prompt?
2. A deny rule and an allow rule both match one request. Which wins, and what is the rule that makes this true?
3. Why must the check use the resolved path, not the path the model wrote?
4. A user answers "always" to a write. Which scope is the default, and what should `/permissions` show afterwards?
5. A denied call returns no result to the model. What goes wrong on the next request?
6. Why are the modes data, and not separate code paths?

---

## Answer key

1. The model can be persuaded by text in a document, and a prompt rule can be ignored. Code checks the tool and the target, and cannot be argued with.
2. Deny wins. The evaluation order checks all deny rules before any allow rule, so a deny cannot be overridden by an allow.
3. A path such as `..\..\x` looks inside the project as text but points outside once resolved. Only the resolved path shows the real target.
4. The default is "this request only", which saves nothing. `/permissions` should show any session or project rule created by "always", with a way to remove it.
5. Each tool call must be answered by a tool result, or the conversation sent to the model is invalid. The next request may be rejected or confused.
6. Data can be changed without changing code, the same checks apply to every mode, and a fix in the check applies to all modes at once.

---

## Scoring

- 5 or 6 correct: ready for Lesson 7.
- 3 or 4 correct: re-read sections 3.3 and 3.7 of the README, then repeat Exercise 1 and Exercise 5.
- Fewer than 3: repeat Exercises 1 to 4.
