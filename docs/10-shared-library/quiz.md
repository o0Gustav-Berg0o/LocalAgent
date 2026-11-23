# Lesson 10 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why did the project need this refactor now, and not after Lesson 2?
2. Why does copying the search setup into a second project cause problems over time?
3. Why should the console app reference the library, and not the other way around?
4. What is the difference between a shared library and a running service, and when would you pick the service?
5. Why must `/eval` give the same numbers before and after a refactor?
6. Why is `NoteTool` kept in the app, while the search logic it calls moves to the library?
7. What is a good test for whether a piece of code belongs in the shared library?

## Answers

1. Up to Lesson 9 only one entry point needed the search. Lesson 11 adds a second process that needs the same setup, so the duplication now has a second user and has already caused failures. Before that, extracting would have guessed at an interface with no second caller.
2. Each copy drifts. A fix or a tuning change made in one copy is missed in the other, so the two apps quietly return different results for the same question.
3. The library is the reusable part and must not know about any particular app. If it referenced the console app, it would be tied to one frontend and could not be used by a second one.
4. A library is compiled into each app, so nothing has to be running. A service keeps one copy of the index that several clients share, but every client needs that service running first. Pick a service when several clients must see one live index that changes while they run.
5. The refactor should only move code. If the numbers change, the move changed behavior, and you need to find out why before going on. Without a before-and-after number, you can't tell a safe refactor from a broken one.
6. `NoteTool` is an adapter: it describes the search to the model and formats the result as text. That is specific to the agent, so it stays in the app. The search itself knows nothing about the model, so a second frontend can use it directly.
7. If a second frontend would need the code, it belongs in the library. If it only makes sense for one frontend (a console prompt, an MCP attribute), it stays in that app.
