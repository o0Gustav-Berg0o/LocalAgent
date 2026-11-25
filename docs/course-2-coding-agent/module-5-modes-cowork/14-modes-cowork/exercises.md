# Module 5, Lesson 14 — Exercises: Code and Cowork Modes

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/Cowork/`, with a `ModeCommand` and a `RollbackCommand`. It uses `SnapshotStore` and `AtomicFile` from `Shared/Files/`.
- Start from a clean git state, with Module 4 working.
- Commit the sandbox project. Cowork mode creates checkpoints, and a clean starting point makes them easy to compare.
- Start the chat `llama-server`.

---

## Exercise 0 — Find the boundary of the current loop

**Goal:** know where the loop asks the user today, before adding a gate.

**Do**

1. Ask the agent for a three-step task in the sandbox: "rename the function X to Y, update its test, and run the build". Note each point where it asked you something.
2. Count the questions, and note whether any were about the overall plan or only about a single action.

**Observe**
- Which of your answers were about one action, and which would you have wanted to answer about the whole task? That difference is the reason for cowork mode.

**Checkpoint:** a count of the questions, and a sentence on what you would have approved in advance if you could.

---

## Exercise 1 — The mode switch and the badge

**Goal:** the user switches mode with a command, and the status line always shows the current mode.

**Do**

1. Create `Features/Cowork/AgentMode.cs` with an enum: `Code`, `Cowork`. The current mode is held by the session, not by the config.
2. Add `/mode code` and `/mode cowork`, and `/mode` alone to show the current mode and what it means in one line.
3. Show the mode in the status line, as a word, not only a colour. Keep it in the status line from Module 2, Lesson 2.
4. Make a mode switch take effect at the next request. A switch during a streaming answer waits until the answer ends (Module 2, Lesson 4, section 4.4).

Start from this shell:

```csharp
namespace LocalAgent.Features.Cowork;

public enum AgentMode { Code, Cowork }

// Held by the session, never saved in the config.
public sealed class ModeState
{
    public AgentMode Current { get; private set; } = AgentMode.Code;

    // A switch asked for during an answer waits here, and applies at the next request (step 4).
    public AgentMode? Pending { get; private set; }

    public void Request(AgentMode mode) => throw new NotImplementedException(); // students write this

    // Called by the loop before each request. Applies Pending, if any.
    public void ApplyPending() => throw new NotImplementedException(); // students write this

    // One line for /mode: the mode and what it means.
    public string Describe() => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** A mode switch must not change any permission rule. Write a test that switches the mode and checks that the same request gets the same permission outcome in both modes.

**Checkpoint:** switch to cowork and back. The status line shows the mode each time, and a write request in both modes goes through the same permission outcome.

---

## Exercise 2 — The plan parser

**Goal:** a plan file is read with the three rules from the README, and nothing else.

**Do**

1. Create `Features/Cowork/PlanParser.cs`. It takes the text of a plan file and returns a list of steps, each with a number, a text, a done flag, and the line it was on.
2. Apply the three rules: a step line starts with `- [ ]` or `- [x]`, followed by a number and a dot; steps run in number order; other lines are notes and are ignored.
3. Return an error for two steps with the same number, and for a step with no number. Do not guess.

Here is one step, parsed into its parts: done is false, number is 2, text is "Write the new loader with the same fields".

Start from this shell:

```csharp
namespace LocalAgent.Features.Cowork;

// Line is 1-based, so a refusal or a "mark done" can point at the exact line.
public record PlanStep(int Number, string Text, bool Done, int Line);

public static class PlanParser
{
    // The three rules from the README, nothing more. Any error returns no steps:
    // a plan that does not parse does not run. Errors name the line number.
    public static (IReadOnlyList<PlanStep> Steps, IReadOnlyList<string> Errors) Parse(string planText)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** A parser that accepts more than the rules will accept a line a person wrote by mistake, and run it. Keep the rules strict, and test the near misses.

**Checkpoint:** run the parser on these inputs and check the results.

| Input line | Expected |
|---|---|
| `- [ ] 1. Read the loader` | step 1, not done |
| `- [x] 2. Write the loader` | step 2, done |
| `- [ ] Read the loader` | error: no number |
| `* [ ] 3. Test` | ignored (not a checklist line) |
| two lines numbered 4 | error: duplicate number |

---

## Exercise 3 — Propose the plan and gate the start

**Goal:** in cowork mode, the agent writes a plan and does not start until the user approves it.

**Do**

1. When the user asks for a task in cowork mode, the agent writes a plan file to `.agent/plans/<session>.md`. The file is written by the app from the agent's proposal, after a check that the proposal parses (Exercise 2).
2. Show the plan in the console, and ask: "Approve this plan? yes = run it, edit = open the file and wait, no = discard."
3. On yes, start step 1. On edit, wait until the user answers again. On no, discard the plan and return to the prompt.

Start from this shell:

```csharp
namespace LocalAgent.Features.Cowork;

public enum PlanAnswer { Run, Edit, Discard }

public sealed class PlanFile(string path)   // .agent/plans/<session>.md
{
    // Writes the agent's proposal only if PlanParser accepts it. Returns the parse errors, or none.
    public IReadOnlyList<string> WriteProposal(string proposedText) => throw new NotImplementedException(); // students write this

    // Reads the file again each time (Exercise 6). The approved version is what is on disk now.
    public (IReadOnlyList<PlanStep> Steps, IReadOnlyList<string> Errors) Read() => throw new NotImplementedException(); // students write this

    // Changes "- [ ]" to "- [x]" on that step's line only, with AtomicFile (Lesson 3).
    public void MarkDone(PlanStep step) => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** The plan file is written by the app, and the approval is a separate decision. A plan the user has not approved must never run a step. Test the case where the file is edited and then approved: the approved version is the one that runs.

**Checkpoint**
- In cowork mode, a task produces a plan file and a prompt, and no step runs before the answer.
- Answering no leaves no plan running, and the file is marked as discarded (or removed, with a note).

---

## Exercise 4 — Run the steps with gates and checkpoints

**Goal:** each step has a gate before it, and a checkpoint after it.

**Do**

1. Before step N, show the step's text and ask: "Run step N? yes / skip / stop."
2. Run the step through the normal loop. Its actions still go through the permission rules (Lesson 6), and its file changes go through approval (Lessons 7 and 8).
3. After the step, take a checkpoint: snapshot every file the step changed, using the snapshot store from Lesson 8, and record the checkpoint's identifier next to the step.
4. Mark the step done in the plan file, by changing `- [ ]` to `- [x]`. Change only that marker.

Start from this shell:

```csharp
namespace LocalAgent.Features.Cowork;

public enum GateAnswer { Run, Skip, Stop }

public sealed class CoworkRunner(PlanFile plan, SnapshotStore snapshots)
{
    // For each step not done, in number order:
    //   1. Read the plan again. 2. askGate(step). 3. Run the step through the normal loop.
    //   4. Checkpoint the changed files. 5. MarkDone, and write the checkpoint id in a note line.
    // Stops at Stop, at a failed step, or at a stuck step (Exercise 7).
    // The caller passes GatePrompt.Step as askGate, so this file never reads input itself.
    public Task RunAsync(Func<PlanStep, CancellationToken, Task<bool>> runStep, Func<PlanStep, GateAnswer> askGate,
        CancellationToken cancellationToken)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

The two questions are a screen file of their own:

```csharp
// Features/Cowork/GatePrompt.cs: the screen.
namespace LocalAgent.Features.Cowork;

public static class GatePrompt
{
    // "Approve this plan? yes = run it, edit = open the file and wait, no = discard." (Exercise 3)
    public static PlanAnswer Plan(TextReader input, TextWriter output) => throw new NotImplementedException(); // students write this

    // "Run step N? yes / skip / stop." Exact words only. Closed input counts as Stop.
    public static GateAnswer Step(PlanStep step, TextReader input, TextWriter output)
        => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** The "mark done" change is a write to the plan file. Make it through the same atomic write as Lesson 7, so a crash cannot leave the plan half-edited. A plan file that is damaged is a plan that will not run.

**Checkpoint**
- A three-step plan asks three gates. Each step is marked done after it runs.
- Each completed step has a checkpoint identifier in the plan file's note lines.
- Skipping a step leaves it not done and moves on, with a note.

---

## Exercise 5 — Roll back a step

**Goal:** the user can return to the checkpoint before a step, and the plan reflects it.

**Do**

1. Add `/rollback <step number>`. It restores the files from the checkpoint taken before that step, using the undo fingerprint check from Lesson 8.
2. Mark the rolled-back step and every later step as not done in the plan file.
3. Refuse the rollback if any file changed since the checkpoint, and list the files.

Add this to `CoworkRunner`:

```csharp
// Restores the checkpoint taken before stepNumber with SnapshotStore.Undo (Lesson 8),
// for that step and every later done step, newest first. All or nothing.
// On success, marks those steps not done. On refusal, returns the files that changed.
public (bool RolledBack, IReadOnlyList<string> ChangedSince) Rollback(int stepNumber)
{
    throw new NotImplementedException(); // students write this
}
```

**Checkpoint:** run two steps, roll back to step 2, and check that the files match their state before step 2 and that the plan shows steps 2 and later as not done.

---

## Exercise 6 — The plan edited during a run

**Goal:** a change the user makes to the plan is seen at the next gate, and nothing else changes.

**Do**

1. Make the gate read the plan file each time, not once at the start.
2. Test three edits while the plan is paused at a gate: change the text of a future step, delete a future step, and add a step at the end.
3. Check that each edit is respected at the next gate, and that the agent does not modify a step marked done.

**Read this carefully.** The agent may want to add a step. It must not write it into the file. It proposes the step in the console, and the user adds it if they want.

**Checkpoint:** each of the three edits is reflected at the next gate, and a done step is untouched by the agent.

---

## Exercise 7 — Stuck and absent

**Goal:** a step that makes no progress stops, and a gate with no answer waits.

**Do**

1. Count the tool calls within a step. If the count passes a limit (use the per-question limit from Lesson 9 as the model), stop the step, mark it as failed with the reason, and wait at the gate.
2. Test a gate with no answer: leave the prompt waiting for a long time. The app must not proceed on its own.
3. Test a step that fails: make a build fail on purpose in step 2. The plan must stop, and step 3 must not start.

**Checkpoint**
- A stuck step is marked as failed, with its reason, and the plan waits.
- A waiting gate does not proceed.
- A failed step stops the plan at the failure.

---

## Exercise 8 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Mode switch during an answer | type `/mode cowork` while streaming | takes effect after the answer |
| Plan with a vague step | a step that names no file | refused before the first gate, with the reason |
| Plan with twenty steps | a long plan | warned about its length, can still be approved |
| Edit to a done step | change the text of a step marked done | the agent ignores the change, and the file keeps the user's text |
| Rollback with a changed file | edit a file by hand after a checkpoint, then roll back | refused, file listed |
| Plan file damaged | invalid line in the middle of the plan | parser error with line number, nothing runs |
| Shell step in cowork mode | a step that runs the build | the shell rules still apply (Lesson 9) |
| Code mode, same request | the same write in code mode | same permission outcome as cowork |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which mode would you give a student for their first task, and why? Would your answer change for a task that touches six files?
2. A plan step says "update the tests". How would you make that step safe to run, and what would you check before it starts?
3. You approved a plan, and then the agent stopped at step 3. What should you see, and what do you do next?

---

## Discuss

- The plan is a contract between the user and the agent. What does each side promise, and where in the lab is each promise checked?
- Cowork mode is slower than code mode. When is the slowness worth it, and when is it a cost you should not pay?
- A user who approves a whole plan without reading it has done the same thing as approval fatigue (Lesson 6). How would you design the plan prompt to make reading it likely?

---

## Stretch

- Let the plan include a step that asks the user a question, and make the gate wait for the answer.
- Show the plan as a live view that updates as steps finish.
- Let two plans run in sequence, with the second gated on the first. What does a failure in the first do to the second?

---

## Time check

If you are past the planned time, finish Exercises 2, 3, and 4. The parser, the approval gate, and the step checkpoint are the three parts that make cowork mode trustworthy.
