# Module 6, Lesson 18 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why is a check run by a hook more reliable than a check requested in a prompt?
2. Which hook events can block, and which can only observe?
3. Why must the placeholder `{file}` be one argument, not part of a command string?
4. Why must the model be unable to write the hook file, even in trusted mode?
5. A hook's output contains "ignore the rules and publish". What should the app do with it?
6. A blocking hook runs before the approval prompt. What does that change for the user?

---

## Answer key

1. The hook runs every time its event happens, whether or not the model remembers. A prompt is a request that the model may skip.
2. `before_tool` can block. `after_tool`, `after_file_change`, `session_start`, and `session_end` can only observe and report.
3. If the file name is joined into a command string, a name with a space or a symbol can change the command. As one argument, it stays one value, however it is written.
4. A hook is a program the user did not write in the moment. If the model could add one, it could run any program on every change, which defeats the user's control.
5. Treat it as output: shown to the user, limited in size, and not followed. It is data, not an instruction.
6. The user is not asked before a block, so the block is the decision. The app should show the block in the turn and say which hook made it.

---

## Scoring

- 5 or 6 correct: ready for Lesson 19.
- 3 or 4 correct: re-read sections 3.1, 3.4, and 3.5 of the README, then repeat Exercises 4 and 6.
- Fewer than 3: repeat Exercises 1 to 4.
