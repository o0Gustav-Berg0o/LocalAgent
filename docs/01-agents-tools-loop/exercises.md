# Lesson 1 — Exercises

## Exercise 1 — Two trivial tools

**Goal:** get a tool call working end-to-end before anything RAG-related exists.

**Do**
1. Write a `get_datetime()` method that returns the current local date/time as text.
2. Write a `calculate(expression: string)` method that evaluates a simple arithmetic expression and returns the result as text (or an error string — never throw).
3. Attach `[Description]` attributes to both the method and its parameters, describing *when* to use the tool, not just what it does.
4. Register both as tools available to the chat model.

**Observe**
- What JSON schema gets generated from your `[Description]` attributes? (Log or print the tool definitions sent to the model.)

## Exercise 2 — Hand-write the loop

**Goal:** implement the loop yourself, without a framework's automatic function-invocation helper.

**Do**
1. Maintain a conversation history as a list of messages.
2. On each user turn: append the user message, then loop:
   - Call the chat model with the full history + tool definitions.
   - If the response contains tool call requests, execute each one, catch exceptions and turn them into `"Error: ..."` strings, append all results to history, and loop again.
   - If the response is plain text with no tool calls, print it and stop looping.
3. Cap the loop at a small number of iterations (e.g. 6) and print a message if you hit the cap.
4. Add trace output: before each model call, print how many messages are in history; after each tool call, print the tool name, arguments, and (truncated) result.

**Observe**
- Ask: *"What's today's date, and what is 17 * 23?"* How many model calls did it take? Why more than one?
- Ask something that needs no tool at all: *"Hi, who are you?"* Did the model call a tool anyway? Should it have?

**Discuss**
- What would happen if a tool call result were 50,000 characters long? Where would that go, and what would it cost?
- Why does catching tool exceptions and returning them as text matter more here than it would in a typical REST API?
