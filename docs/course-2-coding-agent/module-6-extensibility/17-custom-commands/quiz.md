# Module 6, Lesson 17 — Quiz

Write your own answer to each question first, then check the answer key.

1. What is the difference between a built-in command, a custom command, and a skill?
2. Why must the expansion of a custom command be sent as a user message, not run as a tool call?
3. A custom command named `compact` is in the commands folder. What should the loader do?
4. A template says to switch to code mode and turn off prompts. What happens, and why?
5. Why should the expanded prompt be visible to the user?
6. Why is the argument length limited?

---

## Answer key

1. A built-in command is code in the app. A custom command is a user-written prompt file, run when the user types it. A skill is instructions the model chooses to load. The user decides to run a command; the model decides to load a skill.
2. If the command called a tool directly, it would bypass the loop, the permission check, and the history. Sending the expansion as a user message keeps every action under the same rules.
3. Refuse it and warn, because it would shadow a built-in and confuse the user about which one runs.
4. Nothing changes. The template is text that asks the model, and the mode, the settings, and the rules are code. The model may claim the change, but the status line and the permission prompts show the truth.
5. The user cannot tell whether the answer is about their request or about something the template added. Seeing the expansion makes that clear.
6. A very long argument fills the context, and it may carry text meant to look like instructions. A limit keeps the cost bounded and makes the cut visible.

---

## Scoring

- 5 or 6 correct: ready for Lesson 18.
- 3 or 4 correct: re-read sections 3.1, 3.3, and 3.4 of the README, then repeat Exercises 2 and 5.
- Fewer than 3: repeat Exercises 1 to 5.
