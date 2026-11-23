# Lesson 10 — Separating the Core: a Shared Library

**Slides:** (to be added)
**Quiz:** [quiz.md](quiz.md)

## Learning objectives

- Explain why code should be moved out of an application when a second use appears, and why not earlier.
- Split an application into a reusable core library and a thin frontend.
- Refactor in small steps, using the eval harness from Lesson 8 to prove that behavior did not change.

## Why we are refactoring now

The project started as one console app with one job: answer questions from notes. Each lesson added to it, and the scope grew:

| Lesson | What was added | What it made shared |
|---|---|---|
| 2 | Embeddings, vector store, `/search` | The embedding client and the store |
| 3 | Chunking, cache, `/reindex` | The indexer, which every entry point needs |
| 4 | `search_notes` as a tool | The search logic, now called from the agent loop |
| 8 | `/eval` | The same search path, a second time, to measure it |
| 9 | BM25, RRF, `/search-hybrid` | Another ranking path that must match the first |

By Lesson 9 the search logic existed in several places: `/search`, the `NoteTool` the agent calls, `/eval`, and the hybrid path. Each one builds the query the same way, calls the same store, and applies the same rules. Keeping them in sync by hand is the problem this lesson solves.

The next step makes it urgent. Lesson 11 runs the search tools as a separate process, which needs the same embedding client, indexer, and store. When that was first attempted in our own project, the duplicated setup caused real failures:

- A tool was registered that depended on services the new process never created, so every call failed.
- Setup code was copied from `Program.cs` and only partly updated, leaving unused variables that looked like they worked.
- Console output that was harmless in the console app corrupted the protocol channel of the new process.

None of these were hard bugs. They came from one piece of setup existing in two places. That is the signal to move the shared code once, in a deliberate step, instead of copying it again.

## When extraction is worth it, and when it isn't

Do not extract a library on day one. Code that has one user is easier to change when it sits next to its only caller. A library built before there is a second caller usually guesses the wrong interface, and then it has to be changed as well.

Extract when all of these are true:

1. A second entry point needs the same code now, not someday.
2. The code is stable enough that moving it won't keep changing shape.
3. You can prove the move didn't change behavior, with a test or a harness.

Here, all three hold. Lesson 9 is finished, `/eval` gives a number, and Lesson 11 needs the code now.

## What belongs in the library, and what doesn't

| Goes in the shared library | Stays in the app |
|---|---|
| Chunking, cache, indexer, vector store | Console loop and command dispatch |
| BM25, RRF, query prefix | Agent loop and chat client |
| The search logic that `NoteTool` calls | `NoteTool` itself, the adapter the agent sees |
| One factory method that builds a ready search object for a folder | MCP stdio setup and tool attributes (Lesson 11) |

A useful test: if a second frontend would need the code, it belongs in the library. If it only makes sense for one frontend, such as a console prompt or an MCP attribute, it stays where it is.

`NoteTool` is an adapter. It turns the search into a tool the model can call, with a description and a return format. The search itself has no knowledge of the model, so the search moves and the adapter can stay.

## Why a library instead of a running service

Two shapes can share the search:

1. **A shared library.** Each app references the same project and calls the same factory. Nothing needs to be running in the background. Each process loads its own index, which costs little for a small notes folder, and both read the same `index.json` cache.
2. **A running service.** One process owns the index, and other apps call it over HTTP. There is one copy of the index, but every client needs that service running first.

For this course the library is the default. It has fewer moving parts, and each lesson's code still runs on its own. The service shape is the right choice when several clients must see one live index that changes while they run. That is a real requirement we don't have yet.

## Refactor in small steps

A refactor is safe when every step is small enough to check. The method used in this lesson:

1. Record the baseline (`/eval` and `/search-hybrid`).
2. Move one file or group of files.
3. Build. Fix only errors caused by the move.
4. Run `/eval` and compare with the baseline.
5. Repeat.

If a step changes a number, undo that step and find out why. Do not continue on top of a change you don't understand.

## Common mistakes

- **Moving everything at once.** When `/eval` changes, you can't tell which move caused it.
- **Making everything `public`.** Each public member is a promise to every caller. Make only what the apps use public.
- **Changing behavior while moving code.** A refactor only moves code. Fixing a bug at the same time hides both the bug and the move.
- **Moving the UI with the logic.** Console prompts and MCP attributes don't belong in the library.
- **Extracting before the second use.** If there is only one caller, keep the code where it is.

## What changes, what stays the same

| Stays the same | Changes |
|---|---|
| Search behavior, measured by `/eval` | Namespaces and `using` lines in the apps |
| The cache file format and location | Which project owns each file |
| Tool descriptions (still prompts, still Lesson 5) | The build now has more than one project |

## What you'll build

- A class library project (for example `LocalAgent.Core`) containing the shared code listed above.
- One factory method that takes a notes folder and an embedding generator, and returns a ready search object.
- The console app rewired to reference the library, with `/eval` and `/search-hybrid` giving the same numbers as before the move.
- A short write-up: what moved, what stayed, and how you checked that nothing changed.
