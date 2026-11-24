# Module 2, Lesson 5 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why must a path be resolved before it is checked against the project root, rather than checked as text?
2. A root of `C:\proj` and a request for `C:\proj2\secret.txt`: what should happen, and why does the separator matter?
3. Why does a tool result need a truncation message, and what would the model do without one?
4. A file named `data.txt` contains a zero byte. Should it be read as text? Explain your rule.
5. Why is a three-character minimum useful for a search tool?
6. A description says only "Searches files." Write one sentence that should be added to make the model choose the tool correctly.

---

## Answer key

1. A path such as `..\..\x` looks safe as text but points outside the root once resolved. Links and short names have the same problem. Only the resolved path shows where the file really is.
2. It must be refused. The check must require the root followed by a separator, so that a sibling folder with a longer name that starts with the same letters is not accepted.
3. The model reasons from what it sees. Without the message it may treat partial output as complete and answer confidently from incomplete evidence. The message tells it to narrow the request or ask for more.
4. No. A zero byte is a strong sign of binary content, and the file must be refused with a message. The rule is in the content, not the extension.
5. A very short search matches almost every file, floods the context, and gives the model no useful signal. The minimum makes the tool more useful and cheaper.
6. For example: "Use this to locate a name or message across the project before reading a file. Do not use it to read a whole file; use read_file for that."

---

## Scoring

- 5 or 6 correct: ready for Lesson 6.
- 3 or 4 correct: re-read sections 3.2 and 3.3 of the README, then repeat Exercise 1 and Exercise 3.
- Fewer than 3: repeat Exercises 1 to 5.
