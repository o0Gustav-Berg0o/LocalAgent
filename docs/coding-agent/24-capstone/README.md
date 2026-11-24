# Module 7, Lesson 24 — Capstone: The Agent on a Sample Project, Measured

**Slides:** (to be added)

**Time:** about 6 hours, over one or two sessions. This is the longest lesson in the course, because it asks you to measure what you built, not only to build it.

**Prerequisites:** all lessons 1 to 23 of this course, and Course 1.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Set up a sample project with known tasks, so that the agent's work can be checked.
2. Write a set of coding tasks, each with a check that decides whether the task succeeded, without a person reading the answer.
3. Measure the agent's success rate, its cost in tokens and time, and its refusals, in the two modes.
4. Compare two versions of the agent on the same tasks, and say which change made a difference.
5. Write an honest report: what works, what does not, what is unsafe, and what you would change first.
6. Explain the limits of your measurements, and what they cannot tell you.

---

## 2. Why this lesson exists

The course has taught many pieces: streaming, configuration, permissions, writes, undo, shell, context, memory, sessions, modes, skills, hooks, subagents, MCP, and packaging. Each piece was tested on its own. A capstone asks whether they work together, on a task nobody designed for them.

The most important skill this lesson teaches is measurement. A coding agent is easy to demonstrate and hard to trust. An eval set, the same kind as Course 1's notes harness, turns "it seems to work" into a number that can go up or down. That number is what lets you change the agent with confidence, and what lets you tell a user what the agent can and cannot do.

---

## 3. Concepts

### 3.1 A sample project

The sample project must be small enough to build in seconds, and large enough to need real work. A good size is five to ten source files, a test project with a handful of tests, and one or two deliberate weaknesses: a function with a bug, a missing test, a name that is inconsistent. The project is committed to git, so every run starts from the same state.

The project is the test subject. Keep it separate from the agent's own code, so that an agent change cannot make the project's tests pass by changing the tests.

### 3.2 Tasks with checks

A task is a sentence and a check. The sentence is what the user asks. The check is a command that decides the outcome, with no human judgement:

| Task | Check |
|---|---|
| "Fix the off-by-one error in the paging function" | the project's tests pass, including the new one |
| "Add a test for the empty input case" | the new test exists and passes |
| "Rename the function `Calc` to `Compute` everywhere" | the build passes, and no file contains `Calc(` |
| "Add the missing XML comment on the public method" | a check that the comment exists and the build passes |
| "Remove the unused helper" | the build passes, and the helper's name appears nowhere |

A check must be runnable by a script, and must not depend on the wording of the answer. An answer that says "done" with no change fails the check.

### 3.3 Coverage of the course

A good task set covers each part of the agent. Include at least:

- two read-only questions (answered from the code, checked against a known answer),
- three edit tasks of different sizes,
- one task that needs the shell (a build or a test run),
- one task that should be refused (a write outside the project, or a command outside the allowlist),
- one task that needs a long session, to exercise the budget and compaction,
- one task that uses a skill, if you built skills.

A task set that contains only easy edits measures the easy edits, and nothing else.

### 3.4 What to measure

For each run, record:

- **success:** did the check pass, yes or no,
- **turns and tool calls:** how many, and whether any were refused,
- **tokens:** the peak context size, from the meter,
- **time:** the time to the answer, and the total time,
- **approvals:** how many prompts were shown, and how many were answered yes, no, or always,
- **safety events:** refusals by the path, shell, or host rules, and any prompt that should not have been needed.

Write these in a table, one row per task per run. A spreadsheet or a CSV file is fine.

### 3.5 Comparing two versions

A measurement is useful only as a comparison. Run the same task set on two versions of the agent: before a change and after it. Change one thing at a time. A change that improves the success rate but doubles the tokens may be a bad change; the table shows both.

Keep the sample project in the same state for both runs. Reset it with git, and check that the reset is complete before each run.

### 3.6 The report

The report has five parts:

1. **What the agent did:** the task set and the results, as a table.
2. **What works:** the tasks it reliably completes, with the evidence.
3. **What does not:** the tasks it fails, with the reason, from the record.
4. **What is unsafe:** any safety event that the rules should have caught and did not, or any prompt that should have been shown and was not. Be specific.
5. **What you would change first:** one change, justified by the numbers.

A report that lists only successes is not an honest report.

### 3.7 The limits of the measurement

The eval set is small, and the sample project is not a real codebase. Say so. A success rate of nine out of ten on ten tasks is a useful signal and a weak proof. Also say what the measurement cannot tell you: how the agent behaves on a project you did not design, how it handles a user who changes their mind, and how it behaves on a model you did not test.

---

## 4. Gotchas

**4.1 A check that the agent can pass by cheating.** A check that reads only "the test file exists" passes if the agent creates an empty test. Make checks specific: the test must fail before the change and pass after it.

**4.2 A check that changes the project.** A check that formats the code, and then compares, can pass by formatting. The check must not change the thing it checks.

**4.3 Runs that are not reset.** A run that starts from the state the last run left is measuring a different project. Reset with git before each run, and verify the reset.

**4.4 Measuring the model instead of the agent.** A change in the model changes the results, and a change in the agent may be hidden by it. Fix the model for a comparison, and record its name and the server's context size.

**4.5 Approvals answered by the tester.** If you answer every prompt yes, the approval counts are meaningless. Answer as a user would, and record the choices.

**4.6 A report written before the runs.** Writing the conclusion first and then finding the numbers that support it is the most common way a capstone goes wrong. Write the report from the table.

---

## 5. The lab in outline

You will:

1. set up the sample project, with a clean git state and a reset script,
2. write ten tasks, each with a check,
3. run the task set and record the measurements,
4. change one thing in the agent, and run the set again,
5. compare the two runs, and write the report,
6. present the report, and answer questions about it.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Sample project:** the test subject, committed to git and reset before each run.
- **Task:** a sentence the user asks, with a check that decides the outcome.
- **Check:** a script that decides whether a task succeeded, without a person.
- **Run:** one execution of the task set against one version of the agent.
- **Safety event:** a refusal, or a missing prompt, that the rules should have handled.
- **Report:** the written conclusion, drawn from the table of measurements.

---

## 7. Summary

- Build a sample project, and reset it with git before each run.
- Every task has a check, and the check decides the outcome.
- Measure success, cost, time, approvals, and safety events, in a table.
- Compare two versions with one change at a time.
- Write the report from the table, with what works, what does not, what is unsafe, and what to change first.

Continue to [exercises.md](exercises.md).
