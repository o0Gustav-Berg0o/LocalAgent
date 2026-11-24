# Module 4, Lesson 13 — Sessions: Save, List, Resume

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 4, Lessons 10 to 12: the budget, compaction (which needs the transcript), and project memory.
- Module 2, Lesson 2: complete messages only in the history.
- Module 3, Lesson 8: fingerprints and snapshots.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Define what a session is, and separate it from project memory and from the working history.
2. Write a session file that is append-only, versioned, and readable without the app.
3. Decide what a session must not contain: secrets, large file contents, and state that belongs elsewhere.
4. Resume a session by rebuilding the history from the file, and check that the rebuilt history is valid.
5. Build a session picker that shows enough to choose, and a delete command that asks first.
6. Recover from a damaged session file without losing the rest of it.

---

## 2. Why this lesson exists

Close the app, and the conversation is gone. A user who spends an hour on a task and then loses the thread has to start again, and the agent starts from zero. Project memory (Lesson 12) keeps facts about the project, but not the story of the work: what was tried, what failed, what was decided in which order.

Sessions keep that story. They also give compaction (Lesson 11) something to stand on: the transcript is the record that a summary is checked against. Sessions are therefore not an extra feature. They are where the record lives.

---

## 3. Concepts

### 3.1 Session, history, memory

| | Session | Working history | Project memory |
|---|---|---|---|
| What it is | the saved record of one conversation | the messages sent to the model now | facts about the project |
| Lifetime | until deleted | the current run, compacted over time | until edited |
| Who writes it | the app, as the conversation happens | the app, as a view of the session | the user and the approved agent notes |
| Sent to the model | only what the history holds | yes, as the history | yes, as part of the system prompt |

The working history is a view of the session. After compaction, the history is shorter, and the session file still holds every message.

### 3.2 Append-only

A session file grows by adding lines at the end. It is never edited in place. Each line is one event: a user message, an assistant message, a tool call, a tool result, a compaction, a note about a permission decision. Append-only means:

- a crash can damage only the last line, not the whole file,
- the history of the session can be replayed from the start,
- nothing is lost by a later mistake in the app.

Use one event per line in JSON Lines format: each line is a complete JSON object. A line that cannot be read is skipped, and reported.

### 3.3 Versioning

Every event carries a version number, and the session file has a header that says which version of the format it uses. The reader must handle an older version, and must refuse a newer one with a clear message, rather than guessing what a field means.

### 3.4 What a session must not contain

A session is a record, so it is a place where secrets and large data can accumulate without anyone noticing. A session must not contain:

- keys, tokens, or passwords, even if a tool printed them,
- the full contents of large files: store a reference (path and fingerprint) instead, and keep the text where the file is,
- the output of a command beyond its stored limit (Lesson 9),
- anything the user marked as not to be saved.

The rule for large content is: write the event with a reference, and write the content to a separate file only if the content is needed to replay the session. For most sessions, the path and the fingerprint are enough to show what happened.

### 3.5 Resuming

Resuming reads the session file and rebuilds the working history from its events. Two checks happen before the rebuilt history is used:

1. **Validity:** each tool result must have a matching tool call, and each tool call must have its result. A broken pair is a bug, and the resume must refuse to continue from it, or repair it by marking the missing result as "not recorded".
2. **Current state:** files that the session refers to may have changed since. The resume does not replay file changes. It shows the change in the history as a record, and the next read of a file gets the current text.

### 3.6 Picking a session

A picker shows enough to choose: the date and time of the last change, the first user message (cut to a line), the number of turns, and the project folder. A session with no user message yet is not listed.

The picker must work with a keyboard and with `NO_COLOR` (Module 2, Lesson 4), and must not show session contents that the user did not choose to open.

### 3.7 Deleting

A session is deleted only after a confirmation that names it: "Delete session from 25 Nov, 14 turns, first message 'add tests'? Type yes." Deletion removes the file. It does not remove the project memory, which is separate.

### 3.8 Where sessions live

Sessions live in a folder the app owns, outside the project, for example in the user's application data folder. They are not in the project, so they are not committed by accident. The folder is listed in `/config` and in `/sessions`, so the user knows where to look.

---

## 4. Gotchas

**4.1 A session file that is locked.** Another program, such as a sync client or a backup, can hold the file open. Writes fail. The app must report it, and must not keep the event in memory and lose it at exit.

**4.2 Concurrent sessions.** Two windows of the app can write to the same session. Give each session one writer, and check for an existing lock when a session opens.

**4.3 A session that grows too large.** A session of several thousand turns takes a long time to load. Keep the file, but load only the recent events and the latest compaction summary when resuming, and say so in the picker.

**4.4 The clock.** A session sorted by time depends on a clock that can be wrong or change. Use the time of the last event, and sort by that, not by the file's modification time, which backups can change.

**4.5 Encoding.** Session files contain Swedish text. Write and read as UTF-8 without a byte-order mark, and check it with a test.

**4.6 Deleting the wrong session.** A picker with similar entries invites a mistake. The confirmation must show the details (Section 3.7), and the default answer must be no.

---

## 5. The lab in outline

You will:

1. define the event format and the file header,
2. write events append-only, with one line per event,
3. implement resume with the validity check,
4. build the picker and the delete confirmation,
5. recover from a damaged line,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Session:** the saved record of one conversation.
- **Event:** one entry in the session file.
- **JSON Lines:** a file where each line is a complete JSON object.
- **Append-only:** a file that is only added to, never edited in place.
- **Resume:** rebuilding the working history from a session file.
- **Validity check:** checking that every call has a result before a resume continues.

---

## 7. Summary

- A session is the record; the working history is a view of it.
- The file is append-only, one event per line, versioned, and readable without the app.
- Sessions never hold secrets or large content; they hold references.
- Resume rebuilds the history and checks that every call has a result.
- Deletion needs a confirmation that names the session.

Continue to [exercises.md](exercises.md).
