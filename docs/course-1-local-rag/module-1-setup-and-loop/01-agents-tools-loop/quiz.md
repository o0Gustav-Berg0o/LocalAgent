# Lesson 1 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. In the librarian analogy, which steps does the chat model handle, and which does your code handle?
2. True or false: the model executes tool calls itself. Explain.
3. Why does the agent loop need a maximum iteration cap?
4. What should happen if a tool throws an exception?
5. Why is trace output (printing every model request and tool execution) described as non-negotiable for debugging agents?

## Answers

1. The model handles the language steps (understanding the question, deciding to look something up, summarizing the answer); your code handles the actions on the world (actually opening a file, querying something) — the model can only ever request that action, never perform it.
2. False. The model only emits a structured request (a tool name plus JSON arguments); your code decides whether and how to run it, and what to tell the model happened.
3. Because nothing inherently stops a confused model — especially a small one — from requesting the same tool over and over; the cap is a safety limit so the loop can't run forever.
4. It should be caught and turned into a text error message fed back to the model as the tool's result, not allowed to crash the loop — giving the model a chance to recover or retry.
5. Because if you can't see what the model actually asked for and what your code actually did in response, you're debugging blind — agent behavior is otherwise invisible.
