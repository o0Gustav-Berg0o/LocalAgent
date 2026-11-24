# Module 7, Lesson 24 — Exercises: Capstone

Work in order. Each exercise ends with a **Checkpoint**. This lesson is long; the checkpoints tell you when a part is done.

**Before you start**
- Start from a clean git state of the agent, with Lesson 23 done.
- Choose the model and the server settings, and write them down. They do not change for the comparison.
- Use a separate folder for the sample project, outside the agent's repository.

---

## Exercise 0 — Build the sample project

**Goal:** a small project with a known state, committed to git, with two deliberate weaknesses.

**Do**

1. Create a folder, for example `sample-capstone`, and a small .NET solution in it: a library with five or six source files, and a test project with a few tests.
2. Add two deliberate weaknesses: a bug in one function (for example an off-by-one in a paging helper), and a public method with no test.
3. Commit the project to its own git repository. Tag the commit as `baseline`.
4. Write a reset script that runs `git reset --hard baseline` and `git clean -fd` in the sample folder, and prints the current commit. Run it, and check that the output matches the tag.

**Read this carefully.** The reset must remove everything the agent created, including files it wrote and folders it made. Check with `git status` after the reset. A run that leaves a file behind measures a project that is no longer the baseline.

**Checkpoint:** `dotnet test` in the sample project runs, one test fails because of the bug, and the reset script restores the baseline exactly.

---

## Exercise 1 — Write ten tasks with checks

**Goal:** ten tasks that cover the agent, each with a check a script can run.

**Do**

1. Write the tasks in `tasks.md`. Cover the list in the README (section 3.3): two read questions, three edits of different sizes, one shell task, one refused task, one long-session task, and one skill task if you built skills. Make two more of your own.
2. For each task, write the check as a command or a script, and the expected output of a pass. A check must fail before the agent works and pass after a correct change.
3. Test each check on the baseline: it must fail where a change is needed and pass where no change is needed. Record the result.

**Read this carefully.** A read question's check is the expected answer, written as a string the script can compare with the answer's text, or a specific phrase the answer must contain. Make it strict enough that a vague answer fails.

**Checkpoint:** every task has a check, and each check was tested on the baseline. A check that passes on the baseline without any change is a bad check; fix it.

---

## Exercise 2 — The measurement table

**Goal:** a table that records the measurements for each task in each run.

**Do**

1. Create `results.csv` with columns: run, task, success, turns, tool calls, refused calls, peak context tokens, time to answer, total time, prompts shown, prompts answered yes, no, always, and safety events.
2. Write a small script or a short procedure that fills one row per task, after each run. Fill the context numbers from the meter, and the prompt counts from the decision log.
3. Answer every prompt as a user would, and record your choices. Do not answer everything yes.

**Read this carefully.** The decision log (Module 2, Lesson 6) is the source for prompt counts. If the log is not complete, the table is not complete, and the report cannot be trusted.

**Checkpoint:** one run of the task set produces ten rows, with every column filled.

---

## Exercise 3 — Run the baseline version

**Goal:** the first complete run, on the agent as it is now.

**Do**

1. Reset the sample project. Start the agent in code mode, with the model and settings from the preparation.
2. Run the ten tasks in order, one at a time. Reset the project between tasks, so each task starts from the baseline.
3. Record the results in `results.csv` under the run name "baseline-code".
4. Repeat the run in cowork mode for the tasks that suit cowork mode, and record under "baseline-cowork". Write which tasks you ran in each mode, and why.

**Read this carefully.** Reset between tasks, not only between runs. A task that inherits a change from the task before it measures the combination, not the task.

**Checkpoint:** two runs are recorded, each with its rows. You can state the success rate of each run.

---

## Exercise 4 — Change one thing

**Goal:** a single, deliberate change to the agent, chosen from what the baseline showed.

**Do**

1. Look at the baseline's failures and safety events. Choose one change that targets one failure. Examples: a clearer tool description for the edit tool, a larger context reserve, a shorter shell output limit, or a different default effort level.
2. Write the change and the reason in `CHANGES.md`, before you run anything. Write what you expect to happen to the numbers.
3. Make the change in the agent's repository, commit it, and tag the version `after-change`.

**Read this carefully.** Writing the expectation first is the protection against finding the numbers that support a story. Keep the expectation in `CHANGES.md` and compare it with the result later.

**Checkpoint:** `CHANGES.md` has the change, the reason, and the expectation, and the agent is tagged.

---

## Exercise 5 — Run the changed version

**Goal:** the same task set, on the changed agent, with the same settings and the same reset.

**Do**

1. Reset the sample project. Run the ten tasks in code mode, under the name "after-code". Use the same model and settings as the baseline.
2. Repeat for cowork mode under "after-cowork", on the same tasks as the baseline.
3. Record each row in `results.csv`.

**Checkpoint:** four runs are recorded in total, two per version, with the same tasks and settings.

---

## Exercise 6 — Compare the runs

**Goal:** a comparison that shows what the change did, and what else it did.

**Do**

1. For each task, compare success, tokens, and time between the baseline and the changed version, in the same mode.
2. Count the safety events in each run. A change that adds a safety event is a regression, however good the success rate.
3. Write the comparison as a table: task, baseline result, changed result, and a note.
4. Compare the result with your expectation in `CHANGES.md`. Write whether it matched, and where it did not.

**Read this carefully.** Look at the tasks that did not change, as well as the ones you targeted. A change that improves one task and breaks another is the most useful result a capstone can produce, and the easiest to miss.

**Checkpoint:** a comparison table with every task, and a note on whether your expectation matched.

---

## Exercise 7 — The long-session check

**Goal:** the budget and compaction are exercised by a real long task, and their effect is measured.

**Do**

1. Run the long-session task (a task that reads many files and makes several edits) in both versions.
2. Record the peak context size, the number of compactions, and whether the answer kept the constraints from the start of the session.
3. Write one paragraph on whether compaction helped, with the numbers.

**Checkpoint:** the long-session row has a compaction count, and the paragraph uses the numbers from the table.

---

## Exercise 8 — The report

**Goal:** a written report, drawn from the table, with five parts.

**Do**

1. Write `REPORT.md` with five sections, in this order: what the agent did, what works, what does not, what is unsafe, and what you would change first.
2. For each claim, point to the row or the run in `results.csv` that supports it.
3. In "what is unsafe", list every safety event that the rules should have handled, and every prompt that should have been shown and was not. If there are none, say so, and say how you checked.
4. In "what you would change first", name one change, the number that justifies it, and the expected effect on the table.
5. In "limits", say what the measurement cannot tell you, as in the README (section 3.7).

**Read this carefully.** Write the report from the table, not from memory of the runs. If a sentence in the report has no row behind it, remove the sentence or find the row.

**Checkpoint:** every claim in `REPORT.md` points to a row, and the limits section is not empty.

---

## Exercise 9 — Present the report

**Goal:** explain the agent, the measurements, and the report to a person who did not build it.

**Do**

1. Give a presentation of 15 minutes or less. Show the task set, one row of the table that surprised you, the comparison, and the report's "what is unsafe" section.
2. Answer three questions from the audience. For each, say whether the answer is in the table, in the report, or is an opinion.

**Checkpoint:** the presentation is given, and each question is answered with its source.

---

## Observe (whole lesson)

1. Which task gave the most surprising result? What did it teach you about the agent that the individual lessons did not?
2. Which safety event would have been most harmful on a real project? How close did the rules come to failing?
3. If you had one more week, what would you measure next, and why that rather than something else?

---

## Discuss

- A coding agent that succeeds on ten tasks is not proven safe or useful. What would you need to show before you would let it touch a project you care about?
- The measurements depend on the model, the settings, and the sample project. Which of these would you change first to make the result more general?
- The course taught you to build an agent by hand. What would you now do differently if you were choosing an existing agent to use, and what would you check before trusting it?

---

## Stretch

- Run the same task set with a second model, and compare. Record the model name and the server's context size, and report the difference.
- Add a task set from a different language or project type, and see whether the results hold.
- Publish the sample project, the tasks, and the results, so that another student can run them and check your numbers.

---

## Time check

The capstone is the last lesson, and the most valuable one is the measurement. If you are short of time, do Exercises 0, 1, 3, 5, and 8. A report built from one comparison, with honest limits, is worth more than a report without a comparison.
