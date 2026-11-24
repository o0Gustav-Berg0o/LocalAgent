# Module 6, Lesson 19 — Subagents: Work in a Separate Context

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 4, Lessons 10 and 11: the budget, and compaction as a summary.
- Module 2, Lesson 2: the loop, the roles, and the history rule.
- Module 2, Lesson 6: permissions, which a subagent must not bypass.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why a separate context can make the main agent better at a large task, and what it costs.
2. Define a subagent as a run of the same loop with its own history, its own tool set, and a fixed task.
3. Give a subagent a narrow task and a bounded budget, and return only a short result to the main agent.
4. Restrict a subagent's tools, and show that a restriction is enforced in code, not by a sentence in its prompt.
5. Prevent a subagent from using the permissions the user gave the main agent for a different purpose.
6. Show the subagent's work to the user, as a nested progress view, and decide what the main agent receives.

---

## 2. Why this lesson exists

A large task fills the main agent's context. Reading twenty files to answer one question adds twenty file bodies to the history, and the answer has to share the window with all of them. Compaction (Lesson 11) helps, but it is a lossy summary made after the damage is done.

A subagent works on the question in its own context. It reads the twenty files, reasons about them, and returns a short answer. The main agent's history gets the answer and not the files. The main agent stays small, and the subagent can be as large as its task needs, within its own budget.

The cost is that the subagent may be wrong in a way the main agent cannot see. The lesson is about getting the benefit without losing the ability to check.

---

## 3. Concepts

### 3.1 What a subagent is

A subagent is the same loop, run again, with four differences:

1. **Its own history.** It starts with the task, not the main conversation.
2. **Its own tool set.** It can have fewer tools than the main agent, and never more.
3. **Its own budget.** A limit on the number of tool calls, and on its context size.
4. **A fixed output.** It returns one result, in a set format, and the main agent receives only that.

It is not a second model in a second process, unless you choose that. In this course it is the same provider, with different settings.

### 3.2 Why a narrow task

A subagent works well on a question with a clear answer: "which files call this function?", "does this test cover the error case?", "list the public types in this folder". It works badly on a vague task: "improve the code". A narrow task has an answer the main agent can check, and a vague task produces a summary that cannot be checked.

Write the subagent's task as a question with an expected form of answer. "List the files, one per line, that call `ParseConfig`" is a task. "Look at the config code" is not.

### 3.3 The budget

The subagent's budget is set by the main agent, or by the app's defaults, and it has two parts:

- a limit on tool calls (for example 8),
- a limit on its own context size (for example 4096 tokens, measured by the meter from Lesson 10).

When either limit is reached, the subagent stops and returns what it has, marked as incomplete. A subagent that runs out of budget must say so, not return a confident answer.

### 3.4 Tools are a list, enforced in code

The subagent's tools are a list. The app builds the subagent's tool set from that list, and calls to any other tool are refused by the loop, with a result that says the tool is not available to this subagent. A sentence in the subagent's prompt that says "do not write files" is a request. The list is the rule.

Give read-only subagents read-only tools. A subagent that only needs to search should not have a write tool, even if the write tool is denied by the rules. Removing a tool from the list removes the temptation and the risk.

### 3.5 Permissions do not carry over by default

The user approved some actions for the main agent: "always allow this command for this session" (Lesson 6). A subagent does not inherit those approvals. Its actions go through the rules fresh, and an "always" given to the main agent applies only to the exact request it was given for. A subagent asking for the same command is a new request, and it gets its own prompt, or the rule that already exists.

This is a deliberate choice, because the user approved a request in a context they understood. A subagent with a broader job may ask for things the user never approved.

### 3.6 What comes back

The subagent returns a result in a fixed format: an answer, the evidence (file names and line numbers), a status (complete or incomplete), and a count of the calls it made. The main agent receives this result as a tool result, and the subagent's history is discarded or kept in the session file for the user to read.

The subagent's full history is kept in the session file, so the user can check how the answer was reached. The main agent's history holds only the result.

### 3.7 Showing the work

The user should see that a subagent is running, what it was asked, and when it finished. A nested view works well: the main task on one line, the subagent's steps indented below, and its result marked. A user who sees only the final answer cannot tell whether the subagent did real work.

---

## 4. Gotchas

**4.1 A subagent that returns a confident wrong answer.** The main agent receives the answer and trusts it. Ask the subagent for evidence, and let the main agent check a sample of the evidence.

**4.2 A subagent that calls another subagent.** Recursion can run without end and spend the whole budget. Forbid subagents from starting subagents, or limit the depth to one.

**4.3 A subagent with a copy of the main history.** Copying the full history defeats the purpose: the subagent starts as large as the main agent. Give it the task and the necessary facts only.

**4.4 Parallel subagents writing the same file.** Two subagents that both edit a file produce a conflict the fingerprint check (Lesson 7) will refuse. Keep subagents read-only, or run writing subagents one at a time.

**4.5 A subagent's result that is too long.** A result of several thousand tokens fills the main context. Limit the result's size, and say when it was cut.

**4.6 The user cannot tell which agent acted.** An action in a subagent should be labelled with the subagent's name in the decision log and in the progress view. Otherwise a write looks as if the main agent made it.

---

## 5. The lab in outline

You will:

1. define a subagent as a task, a tool list, and a budget,
2. run a subagent in its own history, and return only its result,
3. enforce the tool list in code and test a refused call,
4. check the budget and mark incomplete results,
5. show the nested progress view and label actions,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Subagent:** a run of the loop with its own history, tools, and budget.
- **Tool list:** the tools a subagent is given, enforced in code.
- **Budget:** a limit on a subagent's tool calls and context size.
- **Incomplete result:** a subagent result marked as stopped before the task was done.
- **Evidence:** the file names and line numbers that support a subagent's answer.
- **Nested view:** a progress display that shows a subagent's steps under the main task.

---

## 7. Summary

- A subagent works in its own context and returns a short result, keeping the main context small.
- Its task is narrow, with an expected form of answer.
- Its tools are a list enforced in code, and its budget stops it cleanly.
- Permissions do not carry over; its actions are checked on their own.
- The user sees its work, and the full history is kept for checking.

Continue to [exercises.md](exercises.md).
