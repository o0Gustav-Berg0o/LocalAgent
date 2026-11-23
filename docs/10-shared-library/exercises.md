# Lesson 10 — Exercises

Both `llama-server` windows must still be running.

**Prerequisites:** Lesson 9 is complete, so `/search-hybrid` runs. Work in a clean git state, so every step can be undone with a single revert.

**Rule for this lesson:** after every move, build, run `/eval`, and compare with the baseline from Exercise 1. If a number changes, undo that step before doing anything else.

## Exercise 1 — Record the baseline

**Goal:** numbers you can compare against after every step.

**Do**

- [ ] `/reindex`, then `/eval`. Write down the total and the full PASS/FAIL list.
- [ ] Run `/eval` a second time without changing anything. Compare the two runs.
- [ ] Run `/search-hybrid WIFI_EVENT_STA_DISCONNECTED` and note the top result and its score.

**Observe**
- If the two `/eval` runs differ with nothing edited, something non-deterministic is in the pipeline. Find it before refactoring, or every later comparison is meaningless.

## Exercise 2 — Create the library project

**Goal:** an empty shared project that compiles on its own.

**Do**

- [ ] Add a new class library project next to the console project (for example `LocalAgent.Core`). Target the same framework as the console app.
- [ ] Add a project reference from the console app to the library.
- [ ] Build the solution. It should compile, because nothing has moved yet.
- [ ] Run `/eval` and compare with Exercise 1.

**Observe**
- Which way does the reference point? The console app depends on the library, never the reverse.

## Exercise 3 — Move the chunking and storage code

**Goal:** the lowest-level code lives in the library first, while the console app still compiles.

**Do**

- [ ] Move the chunker and the cache into the library. Keep the namespace unchanged, so the `using` lines in the console app keep working.
- [ ] Build. Fix only errors caused by the move: missing `using` lines, or types that must become `public`.
- [ ] Run `/reindex` and `/eval`. Compare with Exercise 1.

**Observe**
- Which members did you have to make `public` that were `internal` before? Each one is a promise to other callers. Write down the list.

## Exercise 4 — Move the vector store and the ranking code

**Goal:** the search primitives live in the library.

**Do**

- [ ] Move the vector store, BM25, RRF, and the query prefix into the library, keeping the namespace.
- [ ] Build, then run `/eval` and `/search-hybrid`. Compare with Exercise 1.

**Observe**
- The query prefix is used by both `/search` and `/eval`. Where else does it need to be used, and does the library now give you one place to change it?

## Exercise 5 — Move the search logic, and keep the adapter in the app

**Goal:** the search logic is in the library, and `NoteTool` remains in the app as the tool the model calls.

**Do**

- [ ] Move the search logic that `NoteTool` calls into the library, so the library has a method that takes a query and returns ranked passages.
- [ ] Leave `NoteTool` in the console app. It should call the library method and format the result for the model.
- [ ] Build, then run `/eval` and ask the agent one notes question. Compare with Exercise 1.

**Observe**
- Could the library method be used without knowing about tools or the model? If not, something from the adapter has leaked into the library.

## Exercise 6 — Add the factory method

**Goal:** one method builds a ready search object, so no app repeats the setup.

**Do**

- [ ] In the library, add one method that takes the notes folder and an embedding generator, runs the indexer, and returns the store plus the search object.
- [ ] Replace the setup code in the console app's `Program.cs` with a call to that method.
- [ ] Build, then run `/reindex` and `/eval`. The numbers must match Exercise 1.

**Observe**
- The console app's `Program.cs` should now be shorter. What did it lose, and what did it keep?

## Exercise 7 — Prove nothing changed

**Goal:** evidence, not a feeling, that the refactor is safe.

**Do**

- [ ] Run `/eval` and compare the full PASS/FAIL list with Exercise 1, question by question.
- [ ] Run `/search-hybrid WIFI_EVENT_STA_DISCONNECTED` and compare the top result and score with Exercise 1.
- [ ] Write a short note (five lines or fewer) listing what moved, what stayed in the app, and the two comparisons.

## Discuss

- Why is `/eval` a better check for this refactor than running the app once and seeing that it works?
- The library now has a public surface. Which parts would you keep stable, and which can change freely?
- Lesson 11 will host the same search in a separate process. What do you already have in place for that to work, and what is still missing?
- Did you ever feel tempted to fix something while moving code? What did you do instead, and why?
