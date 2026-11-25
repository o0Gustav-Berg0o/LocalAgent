# Module 7, Lesson 21 — Separating the Core from Its Frontends

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 2 and 6: streaming, cancellation, and the single permission check.
- Module 6, Lessons 18 and 20: hooks and external tools, which both need the core to report events.
- the RAG course, Lesson 10: the shared library, and why it was split.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Draw the boundary between the core (the loop, tools, permissions, context, sessions) and the frontend (console, web page, test harness, MCP server).
2. Define the core's output as a stream of events, and its input as a small set of commands, and say why that is a contract.
3. Keep every decision about what the user sees inside the frontend, and every decision about what the agent may do inside the core.
4. Drive the core from a test with no console, using scripted commands and recorded events.
5. Add a second frontend without changing the core, and prove it by running both.
6. Version the contract, and say what a breaking change to it would cost.

---

## 2. Why this lesson exists

So far the console is the only frontend, and it is mixed into the code: the loop prints text, the permission prompt reads keys, the status line writes to a fixed place on the screen. That works for one console. It fails the moment another frontend is needed. A web page, a test that runs thousands of scripted questions, a Module 5 handoff that runs without a person, or the MCP server from the RAG course, each would need a copy of the loop, or changes to the loop for their own display.

Separation makes the core a library with a contract. The frontends use the contract. The core does not know which frontend is attached.

---

## 3. Concepts

### 3.1 The boundary

| Core (no screen, no keyboard) | Frontend (screen and keyboard) |
|---|---|
| the loop and the history | how text is shown and coloured |
| tools and the permission check | the prompt that asks, and how the answer is read |
| the context budget and compaction | the status line |
| sessions, memory, handoff | the keys (Esc, Tab, arrows) |
| the decision log | the layout and the banner |

A test: if a piece of code would need changing for a different screen, it belongs in the frontend. If it would need changing for a different model or a different rule, it belongs in the core.

In this course the boundary is already in the file names. Since Module 1, Lesson 1, logic files never call `Console`, and screen files end in `Screen`, `Prompt`, `Menu`, `Wizard`, or `Command` (Module 1, Lesson 1, section 3.10). In this lesson the logic files move into `LocalAgent.Core`, with the same folders and namespaces, and the screen files stay in the console app, which becomes the first frontend. The few places that are still mixed, such as the loop's streaming output and the working indicator, are what Exercises 0 to 3 find and split.

### 3.2 Events out

The core reports what happens as a stream of events. Each event is a small, typed record:

- `TextDelta`: a piece of the answer, as it streams,
- `ToolStarted` and `ToolFinished`: a tool call and its result,
- `ApprovalNeeded`: a request that needs a yes or no, with the target and the choices,
- `TurnFinished`: the answer is complete, with the timing,
- `Notice`: something the user should know (budget warning, compaction, stale notes),
- `Error`: a failure, with a message the user can act on.

The frontend subscribes to the stream and decides how to show each event. The core never writes to the screen.

### 3.3 Commands in

The frontend sends commands to the core. A small set is enough:

- `SendMessage(text)`,
- `Answer(approvalId, choice)`,
- `Cancel()`,
- `SetMode(mode)`, `SetSetting(name, value)`,
- `Compact()`, `OpenSession(id)`, and similar.

A command is a request. The core validates it, and may refuse it. A refusal is an event, so the frontend can show it.

### 3.4 The approval is an event and a command

An approval is the clearest case of the contract. The core emits `ApprovalNeeded`, with an identifier and the facts the user needs. The frontend shows the prompt and sends back `Answer(identifier, choice)`. The core waits, and only the answer with the matching identifier counts. An answer for an old or unknown identifier is refused.

This is what makes a second frontend safe: it cannot approve something the core did not ask about.

### 3.5 Why the core must not know the frontend

If the core checks whether it is running in a console, the frontends have leaked into it. The core gives every frontend the same events, and the frontend decides what the user sees. A test harness receives the same events as the console and can assert on them.

### 3.6 Versioning the contract

The events and commands are a contract with every frontend. A change that removes a field or changes its meaning breaks every frontend at once. Rules:

- add fields rather than changing them,
- unknown events are ignored by frontends, not treated as errors,
- a version number is sent when the frontend connects, and the core refuses a frontend that needs a newer version.

A breaking change needs a new version, and the old frontends must be updated or left behind on purpose.

### 3.7 Frontends that cannot wait

A frontend may be slow: a web page on a slow connection, or a console that draws every token. The core must not wait for a frontend to draw before the next token. Events are queued, with a bound, and a frontend that falls far behind is told so, rather than slowing the model.

---

## 4. Gotchas

**4.1 A frontend that reads core state directly.** A console that looks into the history to show the budget is coupled to the core's internals. Send the budget as an event.

**4.2 Commands that bypass validation.** A frontend that sets the mode by writing the config file skips the core's checks. All changes go through a command.

**4.3 Approvals without identifiers.** A core that asks "yes or no?" without an identifier cannot tell which question an answer belongs to. Every approval has a unique identifier.

**4.4 Events that carry secrets.** An event for a tool result carries the result, which may hold a secret (Lesson 12). The core decides what goes in an event; a frontend must not have access to more than the event holds.

**4.5 A test that depends on the screen.** A test that reads the console output is fragile. Test by asserting on events.

**4.6 Cancellation that the frontend cannot send.** A frontend without a cancel command cannot stop a runaway answer. The cancel command is part of the minimum contract, and it must work from every frontend.

---

## 5. The lab in outline

You will:

1. list what the console does today, and mark each piece as core or frontend,
2. define the events and commands as types,
3. move the loop's output to events, and the console's input to commands,
4. move the logic files into `LocalAgent.Core`, one slice at a time, checking `/eval` after each move,
5. drive the core from a test with scripted commands,
6. add a second frontend (a plain test harness or a simple second console), and run both,
7. version the contract and test an old frontend against a new core.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Core:** the part of the app that holds the loop, tools, permissions, budget, and sessions.
- **Frontend:** the part that shows output to a person and receives their input.
- **Event:** a typed record the core emits, which a frontend shows.
- **Command:** a request a frontend sends to the core.
- **Contract:** the set of events and commands, with their meanings and versions.
- **Backpressure:** a limit on how many events can wait for a slow frontend.

---

## 7. Summary

- The core has no screen; the frontend has no decisions about the agent's actions.
- The core emits events, and receives commands, and that is the contract.
- An approval is an event with an identifier and a command with the same identifier.
- The contract is versioned, and changed by adding, not by altering.
- A second frontend, and a test that drives the core, are the proof that the boundary holds.

Continue to [exercises.md](exercises.md).
