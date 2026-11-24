# Module 6, Lesson 18 — Hooks: Reliable Checks Around the Agent

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 5 and 6: the tool calls, the permission check, and the decision log.
- Module 3, Lesson 9: the shell tool, its allowlist, and process trees.
- Module 3, Lesson 7: the write path and the file-change event it produces.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why a check done by code is more reliable than a check requested in a prompt.
2. Define hooks as user-configured commands that run at named events, and list the events the course uses.
3. Run a hook as a process with an argument list, a timeout, and a limit on its output, and read its exit code.
4. Decide, for each event, whether a hook can block the action it guards, and make that decision visible.
5. Prevent the model from installing or changing hooks, so that a hook is always the user's decision.
6. Show hook results inside the turn where they happened, with a clear pass or fail mark.

---

## 2. Why this lesson exists

Lessons 7 to 9 let the agent change files and run commands. After a change, the user often wants a check: format the file, build the project, run the tests that cover it. A prompt can ask the model to do that, and the model does it most of the time. Most is not enough for a check. A check that runs only when the model remembers is not a check.

A hook is a check the code runs, every time the event happens, whether or not the model thinks of it. It is the reliable part of an agent's loop. This lesson builds the mechanism, and the rules that keep it in the user's hands.

---

## 3. Concepts

### 3.1 Events

A hook runs at a named event. The course uses five:

| Event | When it fires | Can it block? |
|---|---|---|
| `before_tool` | before a tool call runs, after the permission check | yes, with a block result |
| `after_tool` | after a tool call finishes | no |
| `after_file_change` | after an approved write or edit is applied | no |
| `session_start` | when a session opens | no |
| `session_end` | when a session closes | no |

A hook that can block is a stronger tool, and it must be used carefully. Most hooks in this course observe and report. Blocking is for checks that protect something, such as refusing a write to a generated folder.

### 3.2 The hook file

Hooks are configured in one file that the user writes, for example `hooks.json` in the app's data folder. Each hook names an event, an optional matcher (a file pattern or a tool name), and a command with its arguments:

```json
{
  "hooks": [
    {
      "event": "after_file_change",
      "match": "*.cs",
      "program": "dotnet",
      "arguments": ["format", "--include", "{file}"],
      "timeoutSeconds": 60,
      "description": "Format the changed C# file"
    }
  ]
}
```

The placeholder `{file}` is replaced with the resolved path of the changed file, as one argument. It is never joined into a string, so a file name with spaces or symbols cannot change the command (Module 3, Lesson 9).

### 3.3 Running a hook

A hook runs like any command: an argument list, a working directory in the project root, a clean environment, a timeout that stops the whole process tree, and an output limit. The exit code decides the result:

- `0`: pass,
- any other code: fail, with the output shown,
- a timeout: fail, with a note that it timed out.

A failed `after_*` hook does not undo the action it followed. It reports. A failed `before_tool` hook that is set to block stops the action and returns the failure to the model.

### 3.4 Hooks are the user's code

A hook runs a program. That is a powerful act, and the rules must reflect it. Three decisions:

1. **The user writes the hook file.** The agent cannot create or change it. The permission rules deny any write to the hook file, whatever the mode (Lesson 6).
2. **The app shows every hook at session start,** so the user sees what will run. The list is shown in `/hooks`.
3. **Hooks run without a prompt for each call,** because they are the user's standing decision, but each run is logged, and the log is visible.

This is a deliberate trade. A hook that prompted every time would be ignored. A hook that is set up once and visible in a list is the user's decision, made in advance.

### 3.5 Hook output

A hook's output can be long, and it can contain text that looks like instructions. Treat it as data (Course 1, Lesson 7), limit it to a set size, and show the last part when cutting, as in Lesson 9. The output appears in the turn where the event happened, so the user sees the check next to the change it checked.

### 3.6 Hooks and the model

The model sees the result of a blocking hook, because the block is a tool result. It does not see the result of an observing hook unless the app adds it to the history. Decide, for each hook, whether the model should see its result, and make the decision visible in `/hooks`. A format hook that the model does not need to see can stay out of the history and save tokens.

---

## 4. Gotchas

**4.1 A hook that runs on every file.** A hook on `*` runs a slow build after every change. Use the narrowest match that is correct, and set a timeout.

**4.2 A hook that edits the file it checks.** A formatter changes the file after the change was approved. The user approved one text and got another. Decide whether formatting is part of the approval, and say so in the prompt, or run the formatter before the review.

**4.3 Hooks that disagree with each other.** Two hooks on the same event may run in an order that changes the result. Run them in the order written in the file, and say so.

**4.4 A hook with a path that escapes.** A hook argument that names a folder outside the project must be refused, like any path (Lesson 5).

**4.5 A hook that prompts for input.** A hook waiting for a keypress holds the session until its timeout. Require hooks to be non-interactive, and list the flags that disable prompts.

**4.6 The model edits the hook file.** If the model can write `hooks.json`, it can add a hook that runs any program. The deny rule in Section 3.4 must be tested, not assumed.

---

## 5. The lab in outline

You will:

1. write a hook file with one `after_file_change` hook,
2. run it as a process with an argument list, a timeout, and an output limit,
3. show hook results inside the turn, with pass and fail marks,
4. make the hook file read-only to the model, and test it,
5. add a blocking `before_tool` hook, and test that it blocks,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Hook:** a user-configured command that runs at a named event.
- **Event:** a moment in the loop where hooks can run.
- **Observing hook:** a hook that reports and does not block.
- **Blocking hook:** a hook that can stop the action it guards.
- **Matcher:** the pattern that decides which events a hook applies to.
- **Standing decision:** a choice made once, in advance, and visible.

---

## 7. Summary

- Hooks are checks the code runs at events, so they run every time, not when the model remembers.
- Each hook is a command with an argument list, a timeout, and an output limit.
- Most hooks observe. Blocking is for protective checks.
- The user writes the hook file; the model cannot change it; every hook is shown and logged.
- Hook results appear in the turn where the event happened.

Continue to [exercises.md](exercises.md).
