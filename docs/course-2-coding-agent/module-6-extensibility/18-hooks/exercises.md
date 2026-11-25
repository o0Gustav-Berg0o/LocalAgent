# Module 6, Lesson 18 — Exercises: Hooks

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/Hooks/`. Hooks run through `CommandRunner` in `Shared/Processes/`.
- Start from a clean git state, with Module 3's write and shell tools working.
- Install a formatter or a build tool you can run in the sandbox. A `dotnet build` works.
- Start the chat `llama-server`.

---

## Exercise 0 — Find a check the model forgets

**Goal:** see how often a prompt-only check is skipped, so the lesson has a reason.

**Do**

1. Ask the agent to change a C# file in the sandbox and to build the project afterwards. Do this five times with five different changes.
2. Count how many times the build was run without you asking for it, and how many times it was skipped.

**Checkpoint:** a count out of five.

---

## Exercise 1 — Write the hook file

**Goal:** the user writes one hook in a file, and the app reads it.

**Do**

1. Create `hooks.json` in the main folder (the app's data folder from Module 1, Lesson 1), never in the working folder, with one hook: event `after_file_change`, match `*.cs`, program `dotnet`, arguments `build`, timeout 120 seconds, and a description.
2. Create `Features/Hooks/HookLoader.cs` that reads the file, validates each hook (the event name must be one of the five, the program must not contain a path, the timeout must be in range), and returns the list.
3. Refuse a hook with an unknown event, and warn with its position in the file.

Start from this shell:

```csharp
namespace LocalAgent.Features.Hooks;

// Event is one of the five names in the README. Match is a file pattern or a tool name.
// ModelSeesOutput is false by default (Exercise 3).
public record Hook(string Event, string Match, string Program, string[] Arguments, int TimeoutSeconds,
    string Description, bool ModelSeesOutput = false);

public static class HookLoader
{
    public static readonly string[] Events = [ /* the five event names from the README */ ];

    // Reads hooks.json. A hook with an unknown event, a program with a path, or a timeout out of
    // range is skipped with a warning that gives its position (1-based). The others are returned.
    public static (IReadOnlyList<Hook> Hooks, IReadOnlyList<string> Warnings) Load(string hooksPath)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Validation is the first line of defence. A hook with a program that contains a path (Module 3, Lesson 9, section 4.4) is refused before it can run.

**Checkpoint:** the loader returns one hook, and a file with an unknown event produces a warning and is ignored for that hook only.

---

## Exercise 2 — Run a hook as a process

**Goal:** a hook runs with the same care as a shell command: argument list, working directory, clean environment, timeout, and output limit.

**Do**

1. Create `Features/Hooks/HookRunner.cs`. Reuse the command runner from Module 3, Lesson 9 if you wrote it there. If not, write the same rules: argument list, `UseShellExecute` off, the project root as working directory, a cleared environment, a timeout that stops the process tree, and an output limit.
2. Replace the placeholder `{file}` in each argument with the resolved path of the changed file, as one argument.
3. Return the exit code, the duration, and the output, cut to the limit with the last part kept.

Start from this shell:

```csharp
namespace LocalAgent.Features.Hooks;

public sealed class HookRunner(CommandRunner runner, string projectRoot)
{
    // Replaces "{file}" inside each argument with the resolved path, so the path stays one argument.
    // Runs through CommandRunner (Lesson 9): same argument list, environment, timeout, and tree stop.
    public Task<CommandResult> RunAsync(Hook hook, string? resolvedFile, CancellationToken cancellationToken)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The placeholder is replaced inside one argument, not in the whole command. A path with a space must stay one argument. Test it with a file name that contains a space.

**Checkpoint:** make a change to a `.cs` file in the sandbox. The build runs after the approved change, and the result shows its exit code.

---

## Exercise 3 — Show the result in the turn

**Goal:** the hook's result appears in the same turn as the change, with a clear pass or fail mark.

**Do**

1. After the hook finishes, print one line in the turn: the hook's description, a mark (a check for pass, a cross for fail), and the duration.
2. For a failure, print the last lines of the output, indented.
3. Do not add the output to the model's history unless the hook is marked as one the model should see (Section 3.6 of the README). Decide the default, and write it in `/hooks`.

**Checkpoint:** a passing build shows a pass mark and a duration. A failing build (put a syntax error in the file) shows a fail mark and the last lines of the error.

---

## Exercise 4 — The model cannot change the hook file

**Goal:** the model cannot write or change `hooks.json`, whatever the mode.

**Do**

1. Add a deny rule for writes to the hook file path to the permission rules, in every mode (Lesson 6). The rule matches the resolved path of `hooks.json` in the app's data folder.
2. Ask the agent to add a hook to `hooks.json`, and to write a file that runs `dotnet publish` on every change.
3. The write must be denied, and the reason shown must name the hook file.

**Read this carefully.** This deny rule protects the whole system. If it is missing, the model can add any program as a hook. Test it in read-only, ask, and trusted modes, and check all three.

**Checkpoint:** the write to `hooks.json` is denied in all three modes, with the reason.

---

## Exercise 5 — Show and log every hook

**Goal:** the user can see which hooks exist, and what they did.

**Do**

1. Add `/hooks` to show each hook: its event, its match, its command as a readable line, its timeout, and whether the model sees its output.
2. Log each hook run in the decision log from Lesson 6: the event, the program, the exit code, and the duration. Do not log the output.
3. Show the hook list once at session start, in one line per hook.

**Checkpoint:** `/hooks` shows the hook, and the log shows a line for each run.

---

## Exercise 6 — A blocking hook

**Goal:** a `before_tool` hook can stop a tool call, and the model is told why.

**Do**

1. Add a blocking hook: event `before_tool`, match a tool name (for example `write_file`), program `cmd` with arguments that check for a marker file, and a description. The hook fails when the marker file exists.
2. Create the marker file in the sandbox and ask the agent to write a file. The hook must block the write, and the model must receive a tool result that says the hook blocked it, with the hook's description.
3. Remove the marker file, and the write must proceed after the usual approval.

**Read this carefully.** A blocking hook runs before the approval prompt, so a block is not a prompt, and the user is not asked. Decide whether this order is right for your app, and write the reason in `/hooks`.

**Checkpoint:** with the marker file present, the write is blocked and the model says so. With the marker removed, the write goes through the approval as usual.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| File name with a space | change `my file.cs` | argument stays one value, hook runs |
| Hook that never ends | a program that sleeps | stopped at timeout, tree gone, fail mark |
| Hook that prints a lot | a program that prints 100,000 lines | output cut, last lines shown |
| Hook path outside root | a program with a path | refused at load |
| Unknown event name | `after_save` | warned, that hook ignored |
| Hook that prompts for input | a program that waits for a key | stopped at timeout |
| Two hooks on one event | two `after_file_change` hooks | run in file order, both results shown |
| Hook output with instructions | output that says "ignore the rules" | shown as output, not followed |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Compare the count from Exercise 0 with a run where the hook does the check. What did the hook change in how often the check happened?
2. A hook can block an action before the user is asked. Is that a good design? What would you require before allowing it?
3. A formatter hook changes the file after the user approved the change. What should happen, and where would you show it?

---

## Discuss

- A hook is the user's decision made in advance. What does the user have to trust, and how does the app make that trust visible?
- Why must the model be unable to write the hook file, even in trusted mode? Who would be harmed if it could?
- Observing hooks and blocking hooks have different risks. How would you explain the difference to a user who wants to block everything?

---

## Stretch

- Add a hook event for a failed tool call, and decide whether the model should see that failure.
- Let a hook declare that it is slow, so the app shows a spinner while it runs (Lesson 8).
- Add a per-hook run count to `/hooks`, and show hooks that have never run.

---

## Time check

If you are past the planned time, finish Exercises 1, 2, 4, and 7. The argument-list runner and the rule that protects the hook file are the two parts that keep hooks from becoming a backdoor.
