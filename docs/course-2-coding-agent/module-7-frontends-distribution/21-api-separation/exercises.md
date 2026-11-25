# Module 7, Lesson 21 — Exercises: Separating the Core

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** the contract goes in `LocalAgent.Core/Shared/Contract/`. In Exercise 4, the logic files move into `LocalAgent.Core`, with the same folders and namespaces, and the screen files stay in the console app.
- Start from a clean git state, with Module 6 working.
- Keep a copy of the working app before you begin, so you can compare behaviour after each move.
- Start the chat `llama-server`.

---

## Exercise 0 — Draw the current boundary

**Goal:** know what the console does today, and which parts are mixed.

**Do**

1. List every place where the code writes to the screen (`Console.Write`, colour changes, cursor moves).
2. List every place where the code reads from the keyboard.
3. For each place, write "core" or "frontend", and if it is mixed (a decision mixed with a display), write both halves.

**Observe**
- Which mixed place was hardest to split? What did it decide and what did it show?

**Checkpoint:** a list of at least ten places, each marked core, frontend, or mixed.

---

## Exercise 1 — Define the events and commands

**Goal:** the contract exists as code, with types, before anything moves.

**Do**

1. In the `LocalAgent.Core` project, create `Shared/Contract/Events.cs` with the events from the README: text delta, tool started, tool finished, approval needed, turn finished, notice, and error. Each carries only the data a frontend needs to show it.
2. Create `Shared/Contract/Commands.cs`, in the same folder, with the commands: send message, answer, cancel, set mode, set setting, compact, and open session.
3. Add a version constant to the contract, and a comment that says what a breaking change means.

Here are two events, as orientation. They share a base type, so a sink takes one parameter:

```csharp
namespace LocalAgent.Shared.Contract;

public static class ContractVersion
{
    // A breaking change (a removed or renamed field, a changed meaning) raises this number.
    public const int Value = 1;
}

public abstract record AgentEvent;

// A piece of the answer, as it streams. Frontends append it to their display.
public sealed record TextDelta(string Text) : AgentEvent;

// A request that needs a person. The identifier ties the answer back to this request.
public sealed record ApprovalNeeded(string ApprovalId, string Tool, string Target, string Summary, IReadOnlyList<string> Choices) : AgentEvent;
```

The rest follow the same pattern. Here are their names; the fields are yours to decide, from what a frontend needs to show:

```csharp
public sealed record ToolStarted(/* the call id, the tool, a short target */) : AgentEvent;
public sealed record ToolFinished(/* the call id, ok or not, a one-line summary, the duration */) : AgentEvent;
public sealed record TurnFinished(/* the timing from Module 2, Lesson 2 */) : AgentEvent;
public sealed record Notice(/* the text, and a level */) : AgentEvent;
public sealed record AgentError(/* the text a user can act on */) : AgentEvent;

// In Commands.cs, the same pattern with its own base type:
public abstract record AgentCommand;
public sealed record SendMessage(string Text) : AgentCommand;
public sealed record Answer(string ApprovalId, string Choice) : AgentCommand;
public sealed record Cancel() : AgentCommand;
public sealed record SetMode(/* ... */) : AgentCommand;
public sealed record SetSetting(/* ... */) : AgentCommand;
public sealed record Compact() : AgentCommand;
public sealed record OpenSession(/* ... */) : AgentCommand;
```

The error event is named `AgentError`, not `Error`, so it does not clash with other names in a frontend.

**Read this carefully.** Every event is a record with no behaviour. If an event has a method that draws something, the frontend has leaked into the core.

**Checkpoint:** the contract compiles, and a comment in each file says what the event or command means.

---

## Exercise 2 — The core emits events

**Goal:** the loop reports through events, and no longer writes to the screen.

**Do**

1. Create `Shared/Contract/IEventSink.cs` in `LocalAgent.Core`, with one method that takes an event. The core calls it; the frontend implements it.
2. Change the loop so that each text piece becomes a `TextDelta` event, and each tool call becomes `ToolStarted` and `ToolFinished`.
3. Remove every `Console.Write` from the loop. Do not move them into another class in the core.

The interface:

```csharp
namespace LocalAgent.Shared.Contract;

public interface IEventSink
{
    // Called by the core. It must return quickly: a slow frontend queues events, it does not
    // hold up the loop. It must not throw back into the core.
    void Emit(AgentEvent agentEvent);
}
```

**Read this carefully.** Moving the screen code into a helper inside the core still leaves the core knowing about the screen. The only way out is to emit an event and let a frontend decide.

**Checkpoint:** the loop compiles with no reference to the console. Run the console frontend, and the answer still streams, because the frontend now writes the events.

---

## Exercise 3 — The approval as an event and a command

**Goal:** the permission prompt goes through the contract, with an identifier, and the answer must match.

**Do**

1. When the permission check needs a person, the core creates an identifier, emits `ApprovalNeeded`, and waits for an `Answer` command with the same identifier.
2. An answer with an unknown or old identifier is refused, and the refusal is an event.
3. Move the console's key reading for the prompt into the console frontend. The frontend sends `Answer` with the identifier it was given.

Start from this shell, in the core (`LocalAgent.Core/Shared/Permissions/ApprovalBroker.cs`):

```csharp
namespace LocalAgent.Shared.Permissions;

public sealed class ApprovalBroker(IEventSink sink)
{
    // Creates the identifier, emits ApprovalNeeded, and waits for the matching Answer.
    // Cancel or a closed frontend ends the wait as a refusal.
    public Task<string> RequestAsync(string tool, string target, string summary, IReadOnlyList<string> choices,
        CancellationToken cancellationToken)
    {
        throw new NotImplementedException(); // students write this
    }

    // Called when an Answer command arrives. An unknown or old id is refused: return false,
    // and emit a Notice that says why.
    public bool TryAnswer(Answer answer)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

A `TaskCompletionSource<string>` per open request, kept in a dictionary by id, is one way to write the wait.

**Read this carefully.** The wait is the core's responsibility. The frontend only sends the answer. If the frontend could decide to skip the wait, the approval would be decided by the screen code.

**Checkpoint**
- An approval appears, and the console answers it through the command. The write happens only after the answer.
- A command with a wrong identifier is refused with a `Notice` or an `Error` event.

---

## Exercise 4 — Move the slices into the core

**Goal:** every logic file is in `LocalAgent.Core`, in the same folder and namespace as before. The console app keeps `Program.cs`, `Shared/Commands/` (the slash-command dispatch), and the screen files.

**Background:** the layout from Module 1, Lesson 1, section 3.10 was chosen for this exercise. Logic files never called `Console`, so they move as they are. This is the refactor from the RAG course, Lesson 10, again, with the same method: move, build, compare.

**Do**

1. Record the baseline: run `/eval`, and keep the numbers.
2. Move `Shared/` first, except the parts that belong to the console: `Shared/Commands/` (typed slash commands are console input), `Shared/Prompts/`, `Shared/Download/DownloadConsentPrompt.cs`, and `Shared/OutputStyle.cs`. Keep each file's folder and namespace. Build, run `/eval`, compare.
3. Move `Agent/`, the loop, which has no `Console` call left after Exercise 2. Build, compare.
4. Move one slice at a time, in lesson order. The logic files go to `LocalAgent.Core/Features/<slice>/`. The files whose names end in `Screen`, `Prompt`, `Menu`, `Wizard`, or `Command` stay in `LocalAgent/Features/<slice>/`. Build and compare after each slice.
5. A logic file that does not build in the core still knows the console. Do not add a console reference to the core to make it build. Split the file into a logic part and a screen part, and write its name in your notes.
6. Make the screen files talk to the core through the contract: a slash command sends a command, and a prompt answers an `ApprovalNeeded` with an `Answer` (Exercise 3).

**Read this carefully.** One slice per step. If `/eval` changes after a move, undo that move and find out why before you go on. A move only moves code. A fix made in the same step hides both the fix and the move.

**Checkpoint**
- A search of `LocalAgent.Core` for `Console.` finds nothing.
- Under `LocalAgent/Features/` and `LocalAgent/Shared/`, every file name ends in `Screen`, `Prompt`, `Menu`, `Wizard`, or `Command`, apart from `OutputStyle` and the dispatch files in `Shared/Commands/`.
- `/eval` gives the same numbers as the baseline.
- Your notes list every file you had to split, and why.

---

## Exercise 5 — Drive the core from a test

**Goal:** the core runs with no console. A test sends commands and checks the events.

**Do**

1. Create a test frontend: a class that implements the event sink, records every event in a list, and sends scripted commands.
2. Write a test that sends one message, waits for `TurnFinished`, and checks that a `TextDelta` event was received before it.
3. Write a test that triggers an approval, checks that `ApprovalNeeded` was emitted with the expected target, and answers no. The file must not change.

Start from this shell, in the test project:

```csharp
public sealed class RecordingSink : IEventSink
{
    public List<AgentEvent> Events { get; } = new();

    public void Emit(AgentEvent agentEvent) => throw new NotImplementedException(); // students write this

    // Completes when an event of type T arrives, or fails after the timeout, so a test never hangs.
    public Task<T> WaitForAsync<T>(TimeSpan timeout) where T : AgentEvent
        => throw new NotImplementedException(); // students write this
}
```

Use the fake provider from Module 2, Lesson 3, so the tests need no model server.

**Read this carefully.** A test that reads the console output is fragile. A test that asserts on events is precise, and it runs the same loop the console does.

**Checkpoint:** the two tests pass with no console window open.

---

## Exercise 6 — A second frontend

**Goal:** a second frontend uses the same core, without any change to the core, and both work.

**Do**

1. Create a second frontend. It can be a simple one that writes events as JSON lines to a file, or a second console with different colours. Choose the simplest that proves the point.
2. Run the core with the console, then with the second frontend, on the same question. Compare the events each receives.
3. Confirm that you did not change any file in the core while adding the second frontend. If you had to, write down what forced the change, and fix the contract instead.

**Checkpoint:** the second frontend receives the same events in the same order as the console, and no core file changed.

---

## Exercise 7 — Version the contract

**Goal:** a frontend and a core agree on a version, and a breaking change is visible.

**Do**

1. At connection, the frontend sends its version and the core replies with its version. The core refuses a frontend that is too old for a required event.
2. Add a new optional field to an event. Check that the old frontend still works, and that it ignores the field.
3. Make a breaking change on purpose (rename a field) and check that the old frontend is refused, with a message that names both versions.

**Read this carefully.** Adding is safe; renaming is breaking. Write this rule in the contract's comment, and follow it for every change in the rest of the course.

**Checkpoint:** the optional field change works with the old frontend, and the breaking change is refused with the version message.

---

## Exercise 8 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Slow frontend | a frontend that waits one second per event | the core continues within its bound; the frontend is told it fell behind |
| Frontend disconnects mid-answer | close the second frontend during a stream | the core cancels or finishes the turn, as documented, without crashing |
| Command during a turn | send `SetMode` while the answer streams | takes effect after the turn, as in Module 5, Lesson 14 |
| Two frontends at once | run both at the same time | both receive every event, and an approval is answered by one |
| Unknown event | a frontend that receives an event it does not know | ignored, not an error |
| Answer after cancel | answer an approval after cancelling the turn | refused, the file is unchanged |
| Secret in an event | a tool result with a key-like string | the core's event rules decide what is sent, and the secret check applies |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which mixed place took the most effort to split? What did you learn about the code by splitting it?
2. A frontend can answer an approval it did not show. Is that a risk? How does the identifier limit it?
3. Compare the two frontends side by side. What did the second one show you that the console did not?

---

## Discuss

- The core is now a library with a contract. What does a student gain from that boundary, and what does the boundary cost them to learn?
- A web frontend would run on another machine, possibly. What would have to change in the contract to make that safe?
- Versioning feels like overhead until a frontend breaks. When did it start to matter in your project?

---

## Stretch

- Add an event log, written by a third frontend, that a student can replay later to reproduce a session.
- Expose the contract over a local network socket, and discuss the authentication you would need.
- Write the contract as a document a student can read without the code. Which parts would you describe in words, and which as types?

---

## Time check

If you are past the planned time, finish Exercises 1 to 5. The events, the identifier-matched approval, the move into the core, and the test that drives the core without a screen are the parts the rest of Module 7 relies on.
