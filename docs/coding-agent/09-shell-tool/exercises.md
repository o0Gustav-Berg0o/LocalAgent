# Module 3, Lesson 9 — Exercises: The Shell Tool

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Commit the sandbox project. Install a tool you can build and test with in the sandbox, such as a small .NET console project with one test.
- Use a test command that hangs on purpose for Exercise 3. Create a tiny script that sleeps for 300 seconds, for testing only, and never add it to the allowlist.
- Start the chat `llama-server`.

---

## Exercise 0 — Find out what the agent can run today

**Goal:** know whether any shell execution exists, before adding the tool.

**Do**

1. Search the app for process creation (`Process.Start`, `ProcessStartInfo`).
2. For each match, write whether it runs through a shell, whether it passes a single string or an argument list, and whether it is behind the permission check.
3. Write one sentence on the riskiest path you found.

**Checkpoint:** you have the list, with the answers for each match.

---

## Exercise 1 — The allowlist as configuration

**Goal:** the allowlist lives in the config file, where the user can read and change it, and is validated like every other setting (Lesson 3).

**Do**

1. Add a `shell` section to the config. Give it these fields: a list of entries, a default timeout in seconds, and a default output limit in bytes.
2. Each entry has: the program name, a list of allowed argument patterns, the timeout (optional), and a short description for the user.
3. Start with three entries: `dotnet` with `build` and `test` (and their common flags), and `git` with `status` and `diff` only.

Here is one entry, as it might appear in the file:

```json
{
  "program": "dotnet",
  "allowedArguments": ["build", "test", "build --no-restore", "test --no-restore"],
  "timeoutSeconds": 180,
  "description": "Build and run tests in the project"
}
```

4. Validate the section on load. Refuse an entry whose program contains a path separator (Gotcha 4.4), and warn about an empty pattern list.

**Read this carefully.** Patterns must match whole arguments. `build` allows `dotnet build`, but must not allow `dotnet build-server`. Write a rule for matching, and test the near-miss.

**Checkpoint:** the config loads with the three entries, and a bad entry produces a warning and is ignored.

---

## Exercise 2 — Run a command safely

**Goal:** a command runs from an argument list, in the project root, with a clean environment and a fixed output encoding.

**Do**

1. Create `Tools/Shell/CommandRunner.cs`. It takes the program, the list of arguments, the working directory, and the timeout.
2. Build a `ProcessStartInfo` with the program and each argument added separately. Do not join the arguments into one string.
3. Set the working directory to a folder inside the project root, checked with the path rule of Lesson 5.
4. Clear the environment, then copy only the variables you decided on (the system path, the user profile, and the language settings). Write the list in a comment.
5. Redirect standard output and standard error, and read them with the output encoding set to UTF-8.

Here is the shape of the start-up. The details are yours:

```csharp
var info = new ProcessStartInfo
{
    FileName = program,
    WorkingDirectory = workingDirectory,
    RedirectStandardOutput = true,
    RedirectStandardError = true,
    UseShellExecute = false,      // no shell: arguments are not parsed
    StandardOutputEncoding = Encoding.UTF8,
    StandardErrorEncoding = Encoding.UTF8,
};
foreach (var argument in arguments) info.ArgumentList.Add(argument);   // one value each
info.Environment.Clear();
// copy only the allowed variables into info.Environment
```

**Read this carefully.** `UseShellExecute = false` is what stops the shell from interpreting the arguments. Setting it to `true` brings back the risk of Section 3.2. Also note `ArgumentList`: adding to `Arguments` as one string reintroduces the same risk.

**Checkpoint:** run `dotnet --version` through the runner. The output is correct, and the working directory is the project root. Run `dotnet build` with an argument containing a space, and check that it is treated as one value.

---

## Exercise 3 — Timeout and process tree

**Goal:** a command that hangs is stopped, with everything it started, and the model is told.

**Do**

1. Start the process and wait for it with the timeout. If the timeout passes, stop the process tree, not only the process.
2. Use a method that stops child processes too. On Windows, one way is to run the taskkill command with the tree flag against the process identifier. Check the exact flags for your Windows version before you rely on them.
3. Return a result that says "timed out after N seconds" and includes any output read so far.
4. Test with the hanging script from the preparation. After the timeout, check that no child of the script is still running.

**Read this carefully.** A script that starts a child process is the real test. If the child keeps running after the timeout, the tree stop is wrong. Check with a process list (Task Manager or `tasklist`) before and after.

**Checkpoint**
- The hanging script stops at the timeout, and the result says so.
- After the stop, no child process of the script remains.

---

## Exercise 4 — Results the model can use

**Goal:** the result of a command is a short, plain text that includes the exit code and the output, cut to a limit with a note.

**Do**

1. Create `Tools/Shell/CommandResult.cs` with a method that formats: the command as it was run, the exit code, the duration, and the output.
2. Limit the output to the configured size. When cutting, keep the last part, because errors are usually at the end, and say how many bytes were cut.
3. Combine standard output and standard error in the order they arrived, or show them in two labelled blocks. Pick one and write why.
4. Never show the environment in the result.

**Read this carefully.** Keeping the end of the output is a choice. A compiler error is often the last line, and a model that sees only the first 2000 lines of a 50,000-line build will not see the error at all.

**Checkpoint:** a build with a deliberate compile error returns the exit code, a non-zero number, and the error lines at the end of the output.

---

## Exercise 5 — Connect the allowlist to the permission check

**Goal:** an allowed command runs without a prompt. Any other command asks, and a denied one does not run.

**Do**

1. Make the shell tool ask the permission engine (Lesson 6) with the program and the arguments as the target.
2. Write the mapping: an entry in the allowlist that matches becomes an **allow** rule for that command pattern. Anything that does not match becomes an **ask** rule by default, and a deny rule can be added in the config.
3. Make sure the match uses the whole command, not only the program name. `dotnet build` and `dotnet publish` are different targets.
4. Show the ask prompt for an unlisted command, with the program and the full argument list shown in plain words.

**Read this carefully.** The mapping has a risk: if an allowlist pattern is wider than intended, the permission engine allows more than the user meant. Check each pattern against what the user thinks it allows, and write a sentence for each one.

**Checkpoint**
- `dotnet build` runs without a prompt, and its output appears in the trace.
- `git push` asks, with the full command shown.
- A command denied by a rule does not run, and the model is told why.

---

## Exercise 6 — Count the calls

**Goal:** a question cannot cause an unbounded number of commands.

**Do**

1. Count the shell calls made while answering one question.
2. Set a limit in the config (for example 6). When the limit is reached, refuse further shell calls for this question, with a message to the model that says so.
3. Reset the count when the next question starts.

**Read this carefully.** The limit is per question, not per session. A per-session limit would stop a long, legitimate session after a few builds.

**Checkpoint:** set the limit to 2, ask for three builds in one question, and confirm that the third is refused with the message.

---

## Exercise 7 — Cancel with Esc

**Goal:** Esc during a command stops the process tree and returns a result that says the command was cancelled.

**Do**

1. Connect the cancellation token from Module 2, Lesson 2 to the command runner.
2. On cancel, stop the process tree, as in Exercise 3, and return a result with "cancelled by user" in place of an exit code.
3. Confirm that the model is not told the command succeeded.

**Checkpoint:** start the hanging script, press Esc, and the process tree is gone. The result says cancelled.

---

## Exercise 8 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Argument with a space | a build argument containing a space | treated as one value |
| Argument with `&&` | `dotnet build && del *` as one argument | passed as text, dotnet rejects it, nothing else runs |
| Program with a path | a program name containing `C:\` | refused |
| Near-miss pattern | `dotnet build-server` against pattern `build` | refused or asked, not allowed |
| Hanging command | the sleep script | stopped at timeout, tree gone |
| Large output | a command that prints 100,000 lines | cut, last part shown, cut stated |
| Non-zero exit code | a build with an error | exit code and error lines shown |
| Swedish letters in output | a command that prints `å ä ö` | letters correct |
| Working directory outside root | a request for another folder | refused |
| Denied command | a deny rule for `git push` | not run, reason given |
| Count limit reached | three builds, limit two | third refused with a message |
| Cancel | Esc during the sleep script | tree gone, cancelled result |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which allowlist entry did you almost add, and then reconsider? What was the risk?
2. A user asks you to allow all `dotnet` commands. What do you say, and what would you suggest instead?
3. The model received a timeout result. Describe what it could do next, and which of those actions the rules would still control.

---

## Discuss

- An allowlist asks the user to think in advance. Is that a cost or a benefit? Who pays the cost of forgetting an entry?
- A command can be safe in one project and dangerous in another. Where should that decision live?
- The model can repeat an allowed command many times. What limit protects the user against that, and what limit would be too strict?

---

## Stretch

- Add a `--dry-run` mode that prints the command that would run, without running it, and use it in the lesson to check patterns.
- Run commands in a Windows sandbox or a container for the untrusted cases. What does that cost in setup?
- Add a per-command output summary for long test runs: the number of tests that passed and failed. How would you parse it reliably for one tool?

---

## Time check

If you are past the planned time, finish Exercises 2, 3, and 5. The argument list, the process tree stop, and the link to the permission check are the three things this lesson exists to get right.
