# Module 6, Lesson 19 — Review Checklist

## Specs

- [ ] A subagent is described by a name, a task with an expected answer form, a tool list, and a budget.
- [ ] A spec with an unknown tool or an empty task is refused at validation.

## Running

- [ ] A subagent has its own history, not a copy of the main history.
- [ ] It runs through the same loop, permission check, and output limits as the main agent.
- [ ] The main agent receives only the result record.

## Tool list

- [ ] A call to a tool outside the subagent's list is refused by the subagent's loop.
- [ ] The refusal holds even when the permission rule for that tool is changed for the test.

## Budget

- [ ] A subagent that reaches its tool-call or context limit returns an incomplete result.
- [ ] The main agent shows the incomplete status, and does not present the answer as final.

## Permissions and labels

- [ ] An "always" approval given to the main agent does not carry over to a subagent.
- [ ] Every action in the decision log is labelled with the agent that made it.
- [ ] The progress view shows the subagent's steps under the main task.

## Recursion

- [ ] `run_subagent` is removed from every subagent's tool list in code.
- [ ] A spec that lists it is refused.

## Reflection

- [ ] I compared the main history size with and without a subagent, and recorded the difference.
- [ ] I can say what evidence I would check before trusting a subagent's answer.
