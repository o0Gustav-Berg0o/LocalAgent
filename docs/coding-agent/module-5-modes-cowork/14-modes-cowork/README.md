# Module 5, Lesson 14 — Code Mode and Cowork Mode

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lesson 6: permission rules and modes as presets.
- Module 3, Lessons 7 to 9: write with approval, edit and undo, and the shell tool.
- Module 4, Lesson 13: sessions and the transcript.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain the difference between a mode and a setting, and say which one a mode switch changes.
2. Build code mode, which acts within the permission rules, and cowork mode, which plans first and works in steps.
3. Define a plan file format that a person can read and edit, and that the app can parse with simple rules.
4. Run a plan step by step, with an approval gate before each step, and a checkpoint after it.
5. Let the user change the plan while the agent is working, and handle the change at the next gate.
6. Decide, for each mode, what happens when the user walks away, and what happens when the agent is stuck.

---

## 2. Why this lesson exists

Lessons 5 to 9 gave the agent its hands. Permissions decide which hands it may use, one action at a time. That is the right model for a task the user already understands: "fix this typo", "add this test". For a larger task, such as "migrate the config loader to the new format", the user cannot approve each action in isolation. They need to see the whole plan, agree to it, and watch the work happen in visible steps.

Cowork mode is for that case. Code mode is for the other one. Both use the same tools and the same checks. What differs is when the agent stops and asks, and what the user sees while it works.

---

## 3. Concepts

### 3.1 A mode is not a setting

A setting (Module 2, Lesson 4) persists in the config file and changes what the app is allowed to do, or how it behaves in general. A mode is a per-session choice, switched with a command, and it changes how the loop asks for approval and when it stops.

Switching modes does not change the model, the effort, or the permission rules. It changes the **gate**: the point at which the loop waits for a person.

| | Code mode | Cowork mode |
|---|---|---|
| Plan | none; the agent acts on the request | a plan file, written first and approved |
| Gate | each action that the permission rules mark as ask | each step of the plan, plus the actions the rules mark as ask |
| Pace | as fast as the rules allow | one step at a time, with a checkpoint |
| Best for | small, well-defined tasks | larger tasks the user wants to follow |

### 3.2 The plan file

The plan is a Markdown file in the project's `.agent` folder, named for the session. The format is a checklist, because it is readable without the app and easy to edit:

```markdown
# Plan: migrate config loader

- [ ] 1. Read the current loader and list its fields
- [ ] 2. Write the new loader with the same fields
- [ ] 3. Add a test that loads an old file
- [x] 0. Confirm the sandbox builds (done before the plan)
```

The parser follows three rules, and nothing more:

1. A step is a line that starts with `- [ ]` (not done) or `- [x]` (done), followed by a number and a dot.
2. Steps are run in the order of their numbers.
3. Any other line is a note, read by the person and by the agent, and ignored by the parser.

Simple rules are the point. A parser that understands headings, nesting, and inline code is harder to test, and a user who edits the file by hand will break it in ways the parser does not expect.

### 3.3 The plan is proposed, then approved

In cowork mode the agent does not start work until the plan is approved. The sequence is:

1. The user asks for a task.
2. The agent writes a plan file and shows it in the console.
3. The user approves the plan, edits it, or refuses it.
4. Only then does step 1 begin.

The plan is written by the agent but owned by the user. Any change the user makes to the file is read at the next gate, and the agent must not assume the file is unchanged.

### 3.4 Gates and checkpoints

A **gate** is a point where the loop stops and waits. A **checkpoint** is a saved state the user can return to. In cowork mode:

- before each step, the gate asks: run this step, skip it, or stop,
- after each step, the checkpoint records the state, and the step is marked done in the plan file,
- the actions inside a step still go through the permission rules (Lesson 6), so a step cannot bypass them.

A checkpoint can use the snapshot store from Lesson 8. A checkpoint is a snapshot of every file the step changed, and a rollback to a checkpoint is an undo of that step.

### 3.5 Editing the plan during a run

The user may edit the plan while the agent works. The rule is simple: **the plan is read at each gate, not cached.** If the user changes step 4 while step 2 is running, step 4 uses the new text. If the user deletes a step, it is skipped. If the user adds a step, it is run in its order.

The agent must never edit a step that is marked done, and must never reorder steps on its own.

### 3.6 When the user walks away

Cowork mode is slow when the user is present, and it can be left running. Decide in advance what happens:

- at a gate with no answer after a timeout: wait, do not proceed,
- when a step fails: stop the plan, mark the step as failed with the reason, and wait,
- when the model is stuck (no progress in a step after a set number of tool calls): stop the step, and report.

Waiting is the default. An agent that proceeds when no one answers has turned a gate into a formality.

### 3.7 Code mode is not unsafe

Code mode is not a looser mode of safety. It runs the same permission rules. What it lacks is the plan and the step gates. A student who thinks code mode skips the checks has misread this lesson. The checks run in both modes.

---

## 4. Gotchas

**4.1 A plan that is too vague to run.** "Improve the code" is a bad step. The plan should be written with concrete steps, and the agent should refuse to start a plan with a step it cannot name a file for. Check each step before the first gate.

**4.2 A plan that is too long.** Twenty steps at a gate each is tiring, and users skip gates. Suggest breaking large tasks into several plans, and keep each plan to about ten steps.

**4.3 Steps that depend on each other silently.** Step 3 may assume step 2 created a file. If step 2 fails, step 3 runs on a missing file. Each step's result should say what it produced, and the next step should check it.

**4.4 A step that changes the plan.** The agent may want to add a step, and it must propose the addition, not write it. Additions are written as proposals in the console, and the user adds them to the file.

**4.5 Checkpoints that fill the disk.** A snapshot of every step can be large. Apply the history limit from Lesson 8 to checkpoints too, and say so.

**4.6 The mode badge is missing.** A user who does not know which mode they are in may approve a plan they did not expect. The status line must show the mode at all times.

---

## 5. The lab in outline

You will:

1. implement the mode switch and the mode badge,
2. write the plan parser with its three rules, and test it,
3. write the plan proposal and the approval gate,
4. run the plan step by step, with a gate and a checkpoint per step,
5. handle a plan edited during a run,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Mode:** a per-session choice that changes how the loop asks for approval.
- **Plan file:** the Markdown checklist that lists the steps of a task.
- **Gate:** a point where the loop waits for a person.
- **Checkpoint:** a saved state after a step, that can be returned to.
- **Step:** one numbered item in the plan.
- **Walk away:** a session where the user is not present to answer a gate.

---

## 7. Summary

- A mode changes the gate, not the rules or the settings.
- Code mode acts within the permission rules. Cowork mode plans first and runs in gated steps.
- The plan is a Markdown checklist, parsed with three simple rules.
- The plan is read at each gate, so the user can edit it during a run.
- Waiting is the default when the user is absent or a step fails.

Continue to [exercises.md](exercises.md).
