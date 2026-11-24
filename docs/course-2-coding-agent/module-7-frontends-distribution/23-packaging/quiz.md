# Module 7, Lesson 23 — Quiz

Write your own answer to each question first, then check the answer key.

1. What does a release contain, and what does it deliberately leave out?
2. What is the difference between a framework-dependent build and a self-contained one?
3. Why is a hardcoded developer path the most common leak in a release?
4. A checksum is published with a release. What does it prove, and what does it not prove?
5. Why must the installation guide be tested by someone other than its author?
6. Why should a first run show a clear next step, instead of a configuration dump?

---

## Answer key

1. It contains the application, a clean default config, examples, a README, and checksums. It leaves out the model servers, the model files, the user's notes and sessions, and any key.
2. A framework-dependent build needs the runtime installed on the target machine. A self-contained build includes the runtime, so nothing else needs to be installed, at the cost of a larger file.
3. It makes the release point to a folder that does not exist on another machine, and it exposes the developer's folder structure. The search finds it, but the real fix is moving it into the config.
4. It proves the file is unchanged since the checksum was computed. It does not prove the file is safe, and it does not prove who published it.
5. The author knows the steps without writing them down, so they skip the steps that feel obvious. A reader who starts from nothing finds every gap.
6. A new user needs to know what happened and what to do next. A dump of settings or a stack trace tells them neither, and it ends the first impression.

---

## Scoring

- 5 or 6 correct: ready for Lesson 24.
- 3 or 4 correct: re-read sections 3.1, 3.3, and 3.6 of the README, then repeat Exercises 3 and 6.
- Fewer than 3: repeat Exercises 1 to 5.
