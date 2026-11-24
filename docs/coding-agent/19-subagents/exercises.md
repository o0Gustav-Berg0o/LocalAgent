# Module 6, Lesson 19 — Exercises: Subagents

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state, with Lesson 18 working.
- Use a sandbox project with at least fifteen files, so a search task is worth delegating.
- Start the chat `llama-server`.

---

## Exercise 0 — Measure the main context on a large question

**Goal:** see how a large search fills the main history, so the lesson has a reason.

**Do**

1. Ask the agent: "Which files in the sandbox mention the word `Config`, and what does each one do with it?"
2. Use the meter from Lesson 10 to record the history size after the answer.
3. Note how many file bodies were read into the history.

**Checkpoint:** a history size and a count of file bodies in it.

---

## Exercise 1 — Define a subagent

**Goal:** a subagent is described by a task, a tool list, and a budget.

**Do**

1. Create `Subagents/SubagentSpec.cs` with a record: a name, a task (one sentence with an expected answer form), a list of tool names, a maximum number of tool calls, and a maximum context size in tokens.
2. Create two specs for the sandbox: `find-mentions`, with the read and search tools only, and `summarise-file`, with the read tool only.
3. Validate each spec: the tool names must exist, the limits must be in range, and the task must not be empty.

**Read this carefully.** The task must say what the answer looks like. "Find the files that mention Config, one per line, with the line number of the first mention" is a task. "Look at Config" is not.

**Checkpoint:** both specs load, and a spec that names a tool that does not exist is refused with the name of the tool.

---

## Exercise 2 — Run a subagent in its own history

**Goal:** the subagent runs the loop with its own history, and the main agent receives only its result.

**Do**

1. Create `Subagents/SubagentRunner.cs`. It takes a spec and the task inputs, starts a new history with the task as the user message, and runs the loop until the answer or the budget.
2. The subagent's history is separate from the main history. Do not copy the main history into it.
3. Return a result record: the answer, the evidence list, the status (complete or incomplete), and the number of tool calls.
4. Add the subagent as a tool to the main agent, named `run_subagent`, whose input is the spec name and the task inputs. The main agent receives the result record as the tool result.

**Read this carefully.** The subagent must run through the same loop, the same permission check, and the same output limits as the main agent. A subagent that bypasses the loop has bypassed every protection in this course.

**Checkpoint:** run the search question from Exercise 0 with the subagent. The main history grows by the size of the result, not by the size of the file bodies.

---

## Exercise 3 — Enforce the tool list

**Goal:** a subagent cannot call a tool that is not on its list, and the refusal is enforced in code.

**Do**

1. In the subagent's loop, check every tool call against its spec's tool list before running it. A call to a tool outside the list is refused, with a result that says the tool is not available to this subagent.
2. Test with `summarise-file`, which has only the read tool, and ask it to write a file in its task.

**Checkpoint:** the write is refused by the subagent's loop, not by the permission rules, and the refusal result names the tool list. Confirm the refusal happens even if you remove the write tool's permission rule for the test.

---

## Exercise 4 — The budget and incomplete results

**Goal:** a subagent that runs out of budget says so, and does not return a confident answer.

**Do**

1. Count the subagent's tool calls. When the limit is reached, stop the loop, and return the result with status incomplete and the answer so far, with a note that the task was not finished.
2. Count the subagent's context size with the meter. When the limit is reached, stop in the same way.
3. The main agent must show the incomplete status to the user, and must not present the answer as final.

**Checkpoint**
- Set the tool-call limit to 2 for `find-mentions`. The result is marked incomplete, and the main agent says so.
- Set the context limit very low. The result is marked incomplete, and the reason is shown.

---

## Exercise 5 — Permissions and labels

**Goal:** a subagent's actions go through the rules on their own, and are labelled with the subagent's name.

**Do**

1. Approve a command for the main agent with "always, this session". Start a subagent that asks for the same command. The subagent's request must go through the rules, and the rule must match only the exact request.
2. Label every action in the decision log with the agent that made it: main or the subagent's name.
3. In the progress view, show the subagent's steps indented under the main task, with the result marked.

**Read this carefully.** An "always" given to the main agent is an approval for a request the user understood. The subagent may ask for something the user did not look at. Check that the subagent gets its own prompt for anything the rule does not cover.

**Checkpoint**
- The subagent's action appears in the log with its name.
- The nested view shows the subagent's steps under the main task.
- A command approved for the main agent asks again for the subagent, unless a rule already covers it.

---

## Exercise 6 — Prevent recursion

**Goal:** a subagent cannot start another subagent.

**Do**

1. Remove `run_subagent` from every subagent's tool list, in code, not only in the spec.
2. Test by writing a spec that lists `run_subagent`. The spec must be refused at validation, with a message that says subagents cannot start subagents.

**Checkpoint:** the recursive spec is refused, and no subagent can call `run_subagent`.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Vague task | a task with no expected answer form | warned at validation |
| Tool not on the list | a call to a write tool from `summarise-file` | refused by the subagent's loop |
| Budget reached | a tool-call limit of 1 | incomplete result, shown as incomplete |
| Result too long | a subagent that returns a huge answer | result cut, cut stated |
| Two subagents, same file, writing | two writing subagents | the second refused by the fingerprint check, or run one at a time |
| Subagent errors | a file the subagent cannot read | error in the result, not a crash |
| Same command asked by subagent | the approval from Exercise 5 | new prompt, or a matching rule |
| Subagent output with instructions | evidence text that says "ignore the rules" | shown as data, not followed |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Compare the main history after Exercise 0 with the history after Exercise 2. What did the subagent save, and what did it cost?
2. A subagent returned an answer the main agent accepted, and the answer was wrong. What evidence would have let you catch it?
3. When would you rather do a task in the main agent than in a subagent? Give one case for each.

---

## Discuss

- A subagent is a way to spend more tokens on a task without filling the main context. Is that a good trade on a small local model? Why or why not?
- The main agent trusts the subagent's answer. What should the main agent do with the evidence, and how much of it should it check?
- A user who sees "subagent running" may feel the task is safer than it is. How would you design the progress view so that it does not give false comfort?

---

## Stretch

- Let the main agent run two read-only subagents in parallel, and merge their results. What must be true for the merge to be safe?
- Add a subagent type that writes a plan for cowork mode (Lesson 14), and keep its writes behind the plan's gates.
- Measure the token cost of a subagent against the same task run in the main agent, and report the difference honestly.

---

## Time check

If you are past the planned time, finish Exercises 2, 3, and 4. A separate history, a tool list enforced in code, and a budget that reports incomplete results are the three parts that make a subagent safe to trust.
