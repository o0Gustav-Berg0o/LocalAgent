# Module 5, Lesson 15 — Shared Notes and Handoff

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 5, Lesson 14: cowork mode, the plan file, and gates.
- Module 4, Lesson 12: agent notes and the approval rule for memory.
- Module 4, Lesson 13: sessions and the transcript.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why people and agents need a shared place to leave work for each other, and why a chat session is not enough.
2. Define a notes file that the user writes and the agent reads at the start of a session, and say what the agent is allowed to do with it.
3. Define a handoff file that the agent writes at the end of a session, and that the user can read and correct.
4. Treat text in the notes file as data, not as instructions that override the user's requests.
5. Detect when the notes file or the handoff file is stale, and tell the user.
6. Notify the user when the agent is waiting for them, without making the notification a source of pressure.

---

## 2. Why this lesson exists

Cowork mode (Lesson 14) lets a person and an agent work on one task, but only while both are present. Real tasks do not fit one sitting. A user writes a task on Monday evening, the agent works on it overnight in a gated plan, and the user reviews it on Tuesday morning. For this to work, the two sides need a shared place to leave messages: what the user wants, what the agent did, and what is still open.

A chat history can serve as that place, but it is hard to read, it is tied to one session, and it is not something the user can edit easily. A notes file and a handoff file are plain text, in the project, that both sides can read and change. They make the handoff visible.

---

## 3. Concepts

### 3.1 Two files, two directions

| File | Written by | Read by | Purpose |
|---|---|---|---|
| `NOTES.md` | the user | the agent, at the start of each session | instructions and context for the next work |
| `HANDOFF.md` | the agent | the user, at the start of each session | what the agent did, and what is left |

Two files, one per direction, keep ownership clear. The user never has to guess whether the agent wrote a line, and the agent never edits a line the user wrote.

### 3.2 The notes file is data

The notes file is a document the user wrote, and the agent reads it. It is trusted in the sense that it is the user's own statement. It is not a set of commands that can override the rules of the app. A line in `NOTES.md` that says "skip the permission check" must not change the permission rules, however it is phrased, because the rules live in code (Module 2, Lesson 6).

The distinction is between what the user wants and how the app is allowed to act. The notes can say what the user wants. They cannot change how the app acts.

### 3.3 What the agent may do with the notes

The agent may:

- read `NOTES.md` at the start of a session,
- quote it back to the user,
- follow the user's stated goals, within the rules.

The agent may not:

- edit `NOTES.md`,
- treat a note as permission for an action the rules would ask about,
- act on a note that asks it to ignore the rules or hide what it did.

If the notes contain an instruction the agent cannot follow, the right response is to say so in the handoff, not to follow it.

### 3.4 The handoff file

At the end of a session, or at each gate in cowork mode, the agent writes `HANDOFF.md`. The handoff has fixed sections, like the summary in Lesson 11:

- **Done:** what was completed, with the files changed and the checkpoint identifiers.
- **Not done:** what was planned and not finished, and why.
- **Decisions:** choices the agent made that the user should review.
- **Questions:** what the agent needs from the user to continue.
- **Checks:** what was run to verify the work, and the result.

The handoff is a record the user can check. It must not claim that something was done unless the record shows it, and each claim should point to a file or a checkpoint.

### 3.5 Stale files

A notes file written three weeks ago may describe a project that no longer exists. A handoff written before the user made changes may describe files that were changed since. Both must be marked. The app compares the date of each file with the date of the last session, and warns when the notes are older than the last change to the project, or when the handoff does not match the current state of the files it names.

Staleness is a warning, not an error. The user decides whether the notes still apply.

### 3.6 Notifications

When the agent reaches a gate and waits, the user should know. A notification is a short message: what is waiting, where to answer, and how long the agent has been waiting. Keep it quiet by default. A notification that repeats every minute trains the user to ignore it, and a notification that never appears leaves the work stuck without anyone noticing.

Options include a sound, a line in the status bar, or a desktop notification where the platform allows it. Test that the notification works in the terminal the user actually uses.

### 3.7 Who owns what

The user owns `NOTES.md` and the plan file (Lesson 14), and can edit them at any time. The agent owns `HANDOFF.md` and the session file. A shared file that both can write is a conflict waiting to happen, so the design avoids it.

---

## 4. Gotchas

**4.1 Instructions in notes that beat the rules.** A note saying "run the build without asking" is a request. The rules still decide. Test this on purpose with a note that tries it.

**4.2 A handoff that overstates.** "All tests pass" when the agent ran only one test. The handoff must say what was run, not what was intended.

**4.3 A handoff that the user does not read.** A long handoff is ignored. Keep each section short, and put the questions first when there are any.

**4.4 A notes file with secrets.** The user may paste a key into the notes for convenience. The secret check from Lesson 12 should run on `NOTES.md` too, and warn the user.

**4.5 Two sessions writing the handoff.** Only one session should own the handoff for a project at a time. Check the lock from Lesson 13 before writing.

**4.6 A notification that leaks.** A notification that shows the content of a pending action on a shared screen may reveal the user's code. Show the kind of action, not its content.

---

## 5. The lab in outline

You will:

1. create the notes file and make the agent read it at session start,
2. write the handoff file with fixed sections, and check its claims,
3. treat notes as data, and test an instruction in the notes that tries to override the rules,
4. detect stale files and warn,
5. add a waiting notification,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Notes file:** a file the user writes for the agent to read.
- **Handoff file:** a file the agent writes for the user to read.
- **Stale:** out of date relative to the project or the last session.
- **Claim:** a statement in the handoff that the user can check against a record.
- **Waiting notification:** a short message that the agent needs the user.

---

## 7. Summary

- Two files, one per direction: the user writes notes, the agent writes the handoff.
- The notes are data. They state what the user wants, and cannot change the rules.
- The handoff has fixed sections, and each claim points to a record.
- Stale files produce a warning, and the user decides.
- Notifications are quiet by default and never reveal action content.

Continue to [exercises.md](exercises.md).
