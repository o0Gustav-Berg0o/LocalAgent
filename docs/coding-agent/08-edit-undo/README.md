# Module 3, Lesson 8 — Edit and Undo

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 3, Lesson 7: propose, review, apply, and the fingerprint check.
- Module 2, Lesson 5: resolved paths and output limits.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Design an edit tool that changes a file by replacing exact text, and explain why exact replacement is safer than rewriting the whole file.
2. Enforce that the text to replace occurs exactly once, and refuse with a useful message when it does not.
3. Apply several edits to one file as one change, so that either all apply or none does.
4. Keep a snapshot of every file before each change, and restore it with an undo command.
5. Make undo safe: refuse to restore over a file that changed since the change was made.
6. Bound the undo history, and say what the bound is.

---

## 2. Why this lesson exists

Lesson 7 can replace a whole file. That is precise but expensive. To change one function in a 800-line file, the model must reproduce all 800 lines, and every line it reproduces incorrectly becomes an unreviewed change. A model is far better at naming the text it wants to change than at rewriting the file.

Editing by exact text solves that: the model gives the text to find and the text to replace it with. If the text is not found exactly once, the edit is refused, and the model is told why. Most failures become visible at once instead of hidden in the diff.

Undo is the other half. A user who approved a change and then found a mistake needs a way back. Undo is only trustworthy if it refuses to restore over work the user did after the change.

---

## 3. Concepts

### 3.1 Exact replacement

An edit has three parts: the path, the **old text** (exactly as it appears in the file), and the **new text**. The tool does one thing: find the old text, and replace it with the new text. It does not guess where the text is, and it does not match approximately.

The rule that makes this safe is **uniqueness**. The old text must occur exactly once in the file. Zero matches means the model is looking at a different version or has the text wrong. More than one match means the model has not said which one it means. In both cases, the tool refuses.

### 3.2 Why refusal messages matter

A refusal is a tool result, and the model reads it. A good refusal gives the model what it needs to try again:

- "The text was not found. The file may have changed. Read the file again and use the exact text."
- "The text appears 3 times, at lines 42, 118 and 203. Include more surrounding lines so it matches once."

The second message teaches the model to add context. It is a small piece of teaching inside a tool, and it saves a round trip.

### 3.3 Several edits, one change

A model often needs two or three changes to one file: a new import, a changed method, and a removed line. Applied one at a time, a failure in the middle leaves the file half-changed. The rule: **a set of edits to one file is applied in memory first, and written only if all of them succeed.**

Order matters. Each edit is applied to the result of the previous one, so later edits see earlier changes. Check that each old text is unique at the point where it is applied, not only in the original file.

### 3.4 Snapshots

Before any change is written, the current content of the file is saved as a snapshot. Each snapshot records:

- the file's resolved path,
- the time,
- the fingerprint of the content (Lesson 7),
- the content itself, or a reference to it.

A snapshot is what undo restores. Keep snapshots in a folder the tool owns, outside the project's source folders (for example `.agent/undo/` in the project root, listed in the skip list of Lesson 5 so that searches do not return them).

### 3.5 Undo and the fingerprint

Undo replaces the current file with the snapshot. The same danger as in Lesson 7 applies: if the user changed the file after the agent's change, undo would discard that work.

Before restoring, undo computes the fingerprint of the current file and compares it with the fingerprint recorded **after** the agent's change. If they match, the file is what the agent left, and restoring is safe. If they differ, the user changed the file after the agent did, and undo refuses, showing the difference and asking for a decision.

### 3.6 Bounded history

Snapshots take disk space, and an unbounded history grows forever. Choose a limit, for example the last 50 changes, and say it in the README and in the config. When the limit is reached, the oldest snapshot is removed. The user must be told this happens, because undo beyond the limit is not possible.

### 3.7 Undo is not a version control system

Undo here is a short history for the agent's changes. It is not git. It does not know about commits, branches, or changes made by the user. Teach students to use git for real history, and to treat this undo as a safety net for the last few steps.

---

## 4. Gotchas

**4.1 Whitespace and line endings in the old text.** The model may write a multi-line old text with `\n` where the file has `\r\n`. The match fails, and the model thinks the text is wrong. Normalise line endings when matching, and write back in the file's own style.

**4.2 Tabs and spaces.** Code indentation can differ from what the model remembers. A match that fails on indentation is a common frustration. Show the nearest match in the refusal, with its line number, so the model can correct the indentation.

**4.3 An old text that is valid in two places.** A short old text like `return null;` may match many lines. The uniqueness rule catches it, but the model will need to add context. Make the message say so.

**4.4 Undo of a multi-file change.** A single user approval may cover changes to several files. Undo must restore all of them or none, or the project is left inconsistent. Group snapshots by change, not only by file.

**4.5 A snapshot folder inside the search results.** If the snapshot folder is not skipped, a search returns old code, and the model edits the snapshot instead of the real file. Skip it everywhere.

**4.6 Disk full during a snapshot.** If the snapshot cannot be written, the change must not proceed. A change without its snapshot cannot be undone, and the user was promised undo.

---

## 5. The lab in outline

You will:

1. build the exact-replacement edit with the uniqueness rule and useful refusals,
2. apply several edits to one file as one change,
3. take a snapshot before every write, and record the fingerprint after it,
4. build `/undo` with the fingerprint check,
5. bound the history and report the bound,
6. test the failure cases, including a multi-file undo.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Exact replacement:** changing text by matching it exactly, not approximately.
- **Uniqueness:** the requirement that the old text occurs exactly once.
- **Snapshot:** a saved copy of a file's content before a change.
- **Undo stack:** the ordered list of changes that can be undone.
- **Bounded history:** a history with a maximum size.
- **Normalise:** convert text to one standard form before comparing it.

---

## 7. Summary

- Edits replace exact text, which must occur exactly once. Refusals teach the model how to try again.
- Several edits to one file are applied in memory and written together.
- Every change takes a snapshot, and records the fingerprint after the change.
- Undo restores a snapshot only if the file is still in the state the agent left it.
- The history is bounded, and the bound is stated.

Continue to [exercises.md](exercises.md).
