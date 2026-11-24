# Module 3, Lesson 9 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why is an allowlist safer than a blocklist for a shell tool?
2. What does running a command from an argument list prevent, compared with a single command string?
3. A command starts a child process, and the timeout ends only the parent. What is the problem, and what should happen?
4. Why should the output limit keep the end of the output, not the beginning?
5. The allowlist entry for `dotnet` allows `build`. Would `build-server` be allowed? What rule decides?
6. Why is the count limit for shell calls per question, not per session?

---

## Answer key

1. A blocklist fails when a dangerous command is not on it, and there are too many ways to do damage to list. An allowlist refuses anything unknown, so a new or unexpected command asks instead of running.
2. Without an argument list, a shell parses the string. Spaces, quotes, `&&`, and redirections can change what runs. With an argument list, each argument is one value, and no shell interprets it.
3. The child process keeps running after the timeout, which may hold files, ports, or consume resources. The whole process tree must be stopped, and the result must say so.
4. Errors are usually printed at the end of a long output, such as the last lines of a compiler log. Keeping the beginning would hide the failure.
5. No, if the pattern matches whole arguments. The rule is that every pattern must match the complete argument list, not a prefix of it.
6. A per-session limit would stop a long, legitimate working session after a few builds. A per-question limit stops a runaway loop inside one answer, which is the risk.

---

## Scoring

- 5 or 6 correct: ready for Lesson 10.
- 3 or 4 correct: re-read sections 3.1, 3.2, and 3.5 of the README, then repeat Exercises 2 and 3.
- Fewer than 3: repeat Exercises 1 to 5.
