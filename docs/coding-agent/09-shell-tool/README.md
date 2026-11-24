# Module 3, Lesson 9 — The Shell Tool and Its Allowlist

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lesson 6: permission rules, modes, and the ask prompt.
- Module 3, Lessons 7 and 8: approval, fingerprints, and snapshots.
- Module 2, Lesson 2: cancellation with a key.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why a shell tool needs an allowlist, and why a blocklist of dangerous commands is not enough.
2. Run a program with an argument list instead of a command string, and explain what that prevents.
3. Set a working directory, a timeout, an output limit, and a clean environment for every command.
4. Stop a command and everything it started, when the timeout passes or the user cancels.
5. Turn a command's exit code and output into a result the model can use, and say when the output was cut.
6. Connect the allowlist to the permission rules, so that allowed commands run without a prompt and anything else asks.

---

## 2. Why this lesson exists

A shell tool is the most powerful tool in this course. A model with a shell can build the project, run its tests, format code, install packages, delete files, and send data to the network. It is also the tool most likely to be used wrongly, because a model can produce a command that looks reasonable and does damage.

The lesson does not ban the shell. A coding agent that cannot build or test is limited to reading and guessing. Instead, the lesson gives the shell a narrow, explicit set of commands that the user has agreed to, and asks for everything else.

---

## 3. Concepts

### 3.1 Allowlist versus blocklist

A **blocklist** says: "these commands are dangerous; refuse them." It fails whenever a dangerous command is not on the list. There are endless ways to delete a file, and a blocklist can never be complete.

An **allowlist** says: "these commands are safe; run them without asking. Everything else asks." It fails safe: a new or unknown command is asked about, not run. The cost is that the user must add commands they need, and that is the right cost to pay.

Use an allowlist. Use a blocklist only as an extra check inside the allowed commands, for example to refuse a `--force` flag.

### 3.2 Argument lists, not strings

A command given as one string is parsed by a shell, which interprets spaces, quotes, pipes, and redirections. A file name with a space, or a value with a `;` in it, can change what runs. The safe form is a **program name plus an argument list**, where each argument is one value, and no shell parses the list.

For example, the program `dotnet` with the arguments `build` and `LocalAgent.csproj` is one safe call. The string `dotnet build && del *` would run two programs if passed to a shell. With an argument list, `&&` is just a text argument that `dotnet` will reject.

### 3.3 What an allowed command looks like

An allowlist entry names:

- the **program**, for example `dotnet`,
- the **allowed argument patterns**, for example `build`, `test`, and `format --verify-no-changes`,
- the **working directory**, which must be inside the project root,
- the **timeout**, for example 120 seconds,
- the **output limit**, for example 32 KB.

An entry that allows `dotnet` with any argument is too broad. `dotnet build` and `dotnet test` are reasonable. `dotnet nuget push` is not, and should never be allowed without an explicit decision.

### 3.4 Working directory and environment

A command runs in a known folder: the project root or a subfolder of it. It must not inherit the user's full environment by accident. Some environment variables hold secrets (tokens, keys). A clean environment passes only what the command needs: the system path, the user profile folder, and the language settings. Everything else is removed.

### 3.5 Timeouts and process trees

A command can hang: a test waits for input, a server never stops. A timeout ends the wait. But ending the parent process is not enough: a build tool may have started child processes that keep running. The safe rule is to stop the **whole process tree**, not only the program the model named.

A timeout is a result, not a crash. The model receives "timed out after 120 seconds", and may decide to try again with a different command.

### 3.6 Output as data

The output of a command is text. A command can print instructions, and a test output can contain a line that looks like a command. Treat the output as data (Course 1, Lesson 7), and limit its size. A failing build can print thousands of lines. Return the last part, which usually holds the error, and say how much was cut.

### 3.7 Exit codes

A command's exit code is the clearest signal of success. Zero means success for most tools, and non-zero means failure. Return the code in the result, next to the output, so the model can see the difference between "the test failed" and "the output was short".

### 3.8 Cancellation

The user can cancel a running command with Esc (Module 2, Lesson 2). Cancel must stop the process tree, as with the timeout, and return a result that says the command was cancelled. The model must not be told that a cancelled command succeeded.

---

## 4. Gotchas

**4.1 A command that prompts for input.** A tool that waits for a yes or no will hang until the timeout. Pass the flags that disable prompts (for example a non-interactive flag), and list them in the allowlist entry.

**4.2 A command that writes outside the project.** A build may write to a folder in the user profile. The allowlist does not stop that. Mention it in the README of the allowed command, and consider whether the command needs a sandbox.

**4.3 Path arguments that escape the root.** An argument such as `..\..\other-project` passed to an allowed command is a path, and the path rule of Lesson 5 must apply to it. Check path-like arguments, not only the program name.

**4.4 A program name with a path.** Calling `C:\temp\dotnet.exe` runs a different program with the same name. Resolve the program to a known location, or refuse a program name that contains a folder.

**4.5 Output encoding.** A tool that prints Swedish letters in a legacy code page produces garbled output. Set the output encoding for the child process, and test with a message that contains `å`, `ä`, and `ö`.

**4.6 Zombie processes after a crash.** If the app crashes while a command runs, the command may keep running. Track running processes, and stop them at start-up if they belong to a previous session.

**4.7 Retries that repeat a damaging command.** A model that sees "failed" may try a variant of the same command. The allowlist limits what variants can exist, but the model can still repeat an allowed command many times. Limit the number of shell calls per question, as Lesson 2 limits tool calls.

---

## 5. The lab in outline

You will:

1. write the allowlist as a config section, with patterns and limits,
2. run a command from an argument list, in a fixed working directory, with a clean environment,
3. add a timeout and stop the whole process tree,
4. return the exit code and the cut output as a result,
5. connect the allowlist to the permission check, so allowed commands run and others ask,
6. test the edge cases, including a hanging command and a cancel.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Allowlist:** the list of commands that run without asking.
- **Blocklist:** a list of dangerous commands; not sufficient on its own.
- **Argument list:** a command given as a program and separate values, with no shell parsing.
- **Process tree:** a process and every process it started.
- **Clean environment:** the environment variables passed to a command, reduced to what it needs.
- **Exit code:** the number a program returns when it ends; zero usually means success.

---

## 7. Summary

- An allowlist fails safe; a blocklist does not.
- Commands run from an argument list, in a known folder, with a clean environment.
- Timeouts and cancels stop the whole process tree, and return a clear result.
- Output is data, limited in size, and the cut is reported.
- Allowed commands run without a prompt; everything else asks.

Continue to [exercises.md](exercises.md).
