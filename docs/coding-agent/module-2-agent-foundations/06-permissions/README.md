# Module 2, Lesson 6 — Permissions: Allow, Ask, Deny

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Lessons 2 to 5: the loop streams, the provider is behind a seam, settings persist, and read tools exist.
- Course 1, Lesson 7: confirmed writes and why approval is enforced in code.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why permission checks belong in code between the model and the tool, and never in the prompt.
2. Define rules with three outcomes (allow, ask, deny), and evaluate them in a fixed, documented order.
3. Match rules to tools, paths, and commands, and explain which rule wins when several match.
4. Design permission modes (for example "read only", "ask for changes", "trusted") as named presets over the same rules.
5. Show the user exactly what is being asked, and record every decision in a log.
6. Make "always allow" a choice with a scope (this request, this session, or this project), and a way to undo it.

---

## 2. Why this lesson exists

Lesson 5 gave the model read tools. Lesson 7 will give it a write tool, and Lesson 9 a shell. Each new capability needs a decision about who may use it, and when.

The simplest rule is "ask every time". It is safe and tiring. A user who approves every read will stop reading the prompts, and then approves a write without looking. The goal of this lesson is different: **ask only when the answer matters, and make the question clear when you do ask.**

A permission system also protects against the model's own mistakes. A model may propose a delete it did not mean, or read a file it should not have. The rules catch those cases without relying on the model to behave.

---

## 3. Concepts

### 3.1 Permissions live in code, not in the prompt

A system prompt that says "never delete files" is a request. A model may follow it most of the time, and fail when a document it reads contains a convincing reason to ignore it (Course 1, Lesson 7). A permission check in code cannot be argued with. It sits between the model's request and the tool's execution, and it looks at the tool's name and arguments, not at the model's explanation.

The order is always: **the model asks, the code decides, the tool runs or does not.**

### 3.2 Three outcomes

Every request gets one of three outcomes:

| Outcome | Meaning | Example |
|---|---|---|
| allow | run without asking | reading a file inside the project |
| ask | show the request and wait for yes or no | writing a file (Lesson 7) |
| deny | refuse, and tell the model why | reading `C:\Windows` |

**Deny** is not the same as "ask and expect no". A denied request never reaches the user, and the model receives a clear refusal it can explain.

### 3.3 Evaluation order

When several rules match one request, the order decides the outcome. Use this order, and write it in the config's comment or the README:

1. **Deny rules** win over everything. A deny cannot be overridden by an allow.
2. **Allow rules** come next.
3. **Ask rules** come next.
4. If nothing matches, the **default** applies, and the default is "ask".

Writing the order down matters. Without a fixed order, two rules that match the same request give results that depend on the order they were written in.

### 3.4 What a rule matches

A rule has three parts:

- **Tool:** the tool name, such as `read_file` or `write_file`, or `*` for all.
- **Target:** a path pattern (for writes and reads) or a command pattern (for the shell, Lesson 9), or `*`.
- **Outcome:** allow, ask, or deny.

Example rules, written as text:

```text
deny   write_file  C:\Users\*\.ssh\*
allow  read_file   <project>\**
ask    write_file  <project>\**
deny   *           *.env
```

The `<project>` placeholder is the project root from Lesson 5. The patterns are matched against the **resolved** path, so `..` cannot slip past them.

### 3.5 Permission modes as presets

A mode is a named set of rules, so a user can switch many rules at once:

| Mode | Reads inside project | Writes | Commands | Use it when |
|---|---|---|---|---|
| read-only | allow | deny | deny | exploring code you do not trust |
| ask | allow | ask | ask | the default, for normal work |
| trusted | allow | allow inside project | ask | you know the task and the project |

The modes are presets, not separate code paths. Switching mode changes the rule set, and the same checks run in every mode. This is important: a bug fix in the check applies to all modes at once.

### 3.6 Asking well

When the outcome is "ask", the question must give the user enough to decide:

- the tool name, in plain words ("write a file"),
- the exact target (the full path, not a shortened name),
- the change itself, when there is one (Lesson 7 shows the diff),
- the choices: yes, no, and, where it makes sense, always for this scope.

A question that says "Allow tool call?" is useless. The user cannot tell what they are approving.

### 3.7 Always allow, with a scope

"Always allow" saves the user from repeating a decision. It is also the most common way a permission system is weakened. Make it deliberate:

- **This request only:** the default. Nothing is saved.
- **This session:** the rule lasts until the app closes.
- **This project:** the rule is saved, and shown in a list the user can edit.

Every "always" choice is shown in `/permissions`, with a way to remove it. A user who cannot see their saved rules cannot trust them.

### 3.8 A decision log

Every decision is logged: the request, the rule that matched, the outcome, and whether the user was asked and what they answered. The log is for the user to read, and for you to debug. It must not contain file contents, only names and paths.

---

## 4. Gotchas

**4.1 A deny that is written after an allow.** If the evaluation order is not fixed, a deny can be silently overridden. Test a request that matches both.

**4.2 Patterns matched against the unresolved path.** A rule `allow read_file <project>\**` matches `<project>\..\..\secret.txt` as text. The match must happen on the resolved path, the same rule as in Lesson 5.

**4.3 Case and short names on Windows.** `PROGRA~1` and `Program Files` are the same folder. Matching must be case-insensitive and resolve short names, or a deny rule is bypassed.

**4.4 The default that is too generous.** A default of "allow" for unknown tools means a new tool is usable before anyone decided it should be. The default is "ask", and new tools start there.

**4.5 Always allow that never expires.** A rule saved years ago for one command can apply to a different command today. Store the exact pattern, and show it.

**4.6 A mode switch that does not apply to the current request.** If the user switches to read-only while a write is waiting for approval, the waiting request must be re-evaluated, or refused. A stale approval must not go through.

**4.7 Approval fatigue.** A user who sees many asks will approve them quickly. Keep asks rare, and make each one specific. A design that asks for every read is a design that trains the user to click yes.

---

## 5. The lab in outline

You will:

1. write the evaluation function with the fixed order and test it,
2. define the three default modes as rule sets,
3. make the check run in one place between the model and every tool,
4. write the ask prompt with the target and the choices,
5. add scoped "always allow" and a `/permissions` view,
6. log every decision, and test the edge cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Rule:** a tool, a target pattern, and an outcome.
- **Outcome:** allow, ask, or deny.
- **Evaluation order:** the fixed sequence in which rules are checked.
- **Mode:** a named preset of rules.
- **Scope:** how long an "always" choice lasts: request, session, or project.
- **Approval fatigue:** the habit of approving prompts without reading them.
- **Decision log:** the record of each request and its outcome.

---

## 7. Summary

- Permissions are enforced in code, after the model asks and before the tool runs.
- Three outcomes: allow, ask, deny. The order is deny, allow, ask, then the default of ask.
- Rules match tool, resolved target, and outcome.
- Modes are presets over the same checks.
- Asks state the target and the choices. "Always" has a scope and can be undone.
- Every decision is logged, without file contents.

Continue to [exercises.md](exercises.md).
