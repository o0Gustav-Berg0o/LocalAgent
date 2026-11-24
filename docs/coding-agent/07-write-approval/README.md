# Module 3, Lesson 7 — Write with Approval and Diffs

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 5 and 6: read tools, path resolution, and the permission check.
- Course 1, Lesson 7: the confirmed-write pattern for notes.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Build a write tool that never writes without the permission check approving it.
2. Compute a diff between the current file and the proposed content, and show it in a form a user can read.
3. Explain why a diff must be computed from the file on disk at the moment of approval, not from a copy taken earlier.
4. Write atomically: a write either completes or leaves the original file untouched.
5. Detect that a file changed between the proposal and the approval, and refuse to write over the change.
6. Limit what a write can touch: file size, file type, and the set of folders it may change.

---

## 2. Why this lesson exists

A write is the first action in this course that changes something the user cares about. Reading a file loses nothing. Writing one can destroy work that was never saved anywhere else. Course 1 added a confirmed write for notes, which was a simple case: a new file with a title. Coding changes are different. They modify existing files, they touch several places at once, and a single approved change can hide a mistake that the user only sees later.

The lesson is about making the write **reviewable**: the user sees what will change, approves exactly that, and the change that happens is the change that was approved.

---

## 3. Concepts

### 3.1 Propose, review, apply

A write happens in three steps, and each one is a separate function:

1. **Propose.** The model asks for a change: a path and the new content, or an edit (Lesson 8). The tool computes what the result would be, but does not write it.
2. **Review.** The user sees the diff and decides. The permission check (Lesson 6) decides whether a prompt is needed at all.
3. **Apply.** The change is written, if and only if the approval is still valid.

Keeping the three apart lets you test each one. A proposal can be shown in a test without writing. An apply can be tested with a fake approval.

### 3.2 What a diff is

A diff is the set of lines that differ between two versions of a text. The usual form has three kinds of line:

- **Context:** unchanged lines near a change, shown for orientation.
- **Removed:** a line that exists now and will be gone.
- **Added:** a line that does not exist now and will appear.

A diff is not the same as a summary. "Changed 3 lines" tells the user nothing about what changed. The diff shows it.

### 3.3 Diffs are computed from the file on disk

The diff must be computed against the file as it is on disk at the moment of approval. Consider this sequence:

1. the model proposes a change, and the diff is computed against version A,
2. the user, in another program, saves version B,
3. the user approves the change shown earlier.

If the approved change is applied to version B, the result is a mix that nobody reviewed. The rule: the approval carries a **fingerprint** of the file it was computed against, and the apply step checks the fingerprint before writing. A mismatch means "the file changed; review again".

A fingerprint can be a hash of the content (SHA-256, as in Module 1). It is cheap to compute and exact.

### 3.4 Atomic writes

An atomic write has the same shape as the atomic save in Module 2, Lesson 4 and the download in Module 1:

1. write the new content to a temporary file in the same folder,
2. check it (size, and that it was written completely),
3. move it over the original in one step.

If anything fails before step 3, the original is untouched. Step 3 must be a move within one volume, so it is atomic on the same disk.

### 3.5 What a write may touch

A write tool has limits, and they belong in the config as well as in the code:

- **Folders:** writes are allowed only inside the project root (Lesson 5), and can be narrowed further, for example to `src` only.
- **File types:** a list of extensions the tool may write, such as `.cs`, `.md`, `.json`, `.txt`. Anything else is refused and must be approved through a different, explicit path.
- **Size:** a maximum size for one write, for example 200 KB. A larger change is refused with a message to split it.
- **Creating versus changing:** creating a new file and changing an existing one can have different rules. Make the difference visible in the prompt.

### 3.6 Line endings and encoding

A change that switches a file from `\r\n` to `\n` shows every line as changed in the diff. This is noise, and it hides the real change. Keep the file's existing line endings, and detect the encoding (Lesson 5, section 4.1) before writing. Writing a file in a different encoding than it had can break characters that were correct before.

---

## 4. Gotchas

**4.1 A diff that hides the real change.** A diff with a large context window or one that normalises whitespace can make a real change hard to see. Show the actual lines, unchanged.

**4.2 An approval that outlives the proposal.** If the user approves a change an hour later, the file may have changed in the meantime. The fingerprint check exists for this.

**4.3 Creating a file in a folder that does not exist.** Decide whether the tool creates folders. The safe default is no: a missing folder is refused with a message, so a typo in a path does not create a new folder tree.

**4.4 A write that succeeds but the content is wrong.** The atomic write checks the size, but a truncated or encoding-broken file may still have the right size. Read the file back after the move, and compare its fingerprint with the proposed content.

**4.5 Backups.** A write replaces the old file. Lesson 8 adds undo. Until then, a write that the user does not want is lost. Say so in the prompt for changes to existing files.

**4.6 A prompt that is too long.** A change to a 2000-line file produces a diff of hundreds of lines. Show a summary and a way to see the rest, or refuse changes above a size and suggest splitting.

**4.7 The model proposes many writes at once.** Each write needs its own review. Do not batch them under one "yes" unless the user chose that, and show the list.

---

## 5. The lab in outline

You will:

1. separate propose, review, and apply into three functions,
2. compute a diff and show it readably,
3. fingerprint the file at proposal time and check it at apply time,
4. write atomically and verify the result,
5. enforce folder, type, and size limits,
6. test the edge cases, including a file changed during review.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Propose:** compute the result of a change without writing it.
- **Diff:** the lines removed and added between two versions.
- **Fingerprint:** a hash of a file's content, used to detect change.
- **Atomic write:** a write that either completes fully or leaves the original untouched.
- **Context lines:** unchanged lines shown around a change.
- **Line endings:** the characters that end a line; Windows uses `\r\n`.

---

## 7. Summary

- A write is three steps: propose, review, apply. Each is a separate function.
- A diff shows the actual lines that change, computed from the file on disk.
- The approval carries a fingerprint; apply refuses if the file changed.
- Writes are atomic, verified after the move, and limited by folder, type, and size.
- Line endings and encoding are kept as they were.

Continue to [exercises.md](exercises.md).
