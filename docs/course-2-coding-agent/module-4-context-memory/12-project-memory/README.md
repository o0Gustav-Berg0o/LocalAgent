# Module 4, Lesson 12 — Project Memory

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 4, Lessons 10 and 11: the budget and compaction.
- Module 2, Lesson 4: the settings menu and validation.
- Module 2, Lesson 5: resolved paths and output limits.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain the difference between memory the user writes, memory the agent proposes, and memory that is only in the conversation.
2. Load a project instruction file into the system prompt on every session, and count what it costs on every request.
3. Design a memory file with a size limit, a structure, and a reason for each section.
4. Let the agent propose a note for the memory, and require the user to approve it before it is saved.
5. Show the user which memory is active, where it comes from, and how large it is.
6. Explain why memory must never contain secrets, and how the app keeps them out.

---

## 2. Why this lesson exists

Compaction (Lesson 11) keeps a conversation within the window, but a conversation ends. The next session starts without what was learned in the last one: the project's build command, the preferred style, the files that matter, the mistakes already made. A user who explains the same things every session stops using the tool.

Memory is the answer, and it is also a risk. Anything saved as memory is loaded into every future request, so it costs tokens every time. It is also trusted by the model, so a wrong note or a note that an untrusted document persuaded the agent to write can shape every later answer. Memory must be small, readable, and approved.

---

## 3. Concepts

### 3.1 Three kinds of memory

| Kind | Who writes it | Where it lives | Example |
|---|---|---|---|
| Project instructions | the user, by hand | a file in the project root | "Build with `dotnet build`. Use snake_case for tools." |
| Agent notes | the agent, approved by the user | a file the app owns | "The config parser is in `Config/Loader.cs`." |
| Conversation | both, during a session | the history (Lessons 10 and 11) | "We agreed to skip the tests for now." |

Only the first two survive a session. The third survives only if it was saved as a session (Lesson 13) or written into an agent note.

### 3.2 The project instruction file

A project instruction file is a plain Markdown file in the project root, read at the start of every session and placed in the system prompt. It is the user's file: the agent reads it, but does not write it without approval.

The file has three properties that matter:

- **It costs tokens on every request.** A 3000-token instruction file is paid for on every turn. Keep it short.
- **It is trusted.** The model follows it. An instruction in this file has more authority than the same words in a note the agent read, so it is the user's responsibility to write it carefully.
- **It is visible.** The user can open it, change it, and see exactly what the model was told.

### 3.3 Where memory sits in the request

Memory is part of the system prompt, which is one of the five parts of the context window (Lesson 10). It is placed after the fixed instructions and before the tool descriptions, so the budget counts it and compaction does not summarise it (Lesson 11, section 3.2).

The order inside the system prompt matters less than the fact that it is counted. A memory file that is not counted in the budget will eventually push a request over the window without warning.

### 3.4 Agent notes need approval

An agent note is a fact the agent believes is useful later. It is proposed, not written. The proposal shows the note text, the section it would go into, and the reason. The user answers yes, no, or edit, using the same ask pattern as Lesson 6.

The rule matters because a note is trusted in every future session. A note written because a document in the project said "remember this" is an injection (Course 1, Lesson 7) that survives restarts. The approval step is the defence.

### 3.5 Size and structure

Memory has a size limit, in characters or tokens, set in the config. When the limit is reached, the app does not silently drop notes. It refuses a new note, and shows the user the current size, so they can remove or merge notes.

Use a fixed structure for agent notes, with sections such as: commands, layout, conventions, and known issues. A section keeps a note where it can be found, and makes it easy to remove a whole class of notes.

### 3.6 No secrets

Memory must not contain:

- keys, tokens, passwords, connection strings with credentials,
- personal data the user did not choose to share,
- the contents of files the user has not approved for memory.

The app keeps secrets out in two ways: a pattern check on every proposed note (it refuses text that looks like a key), and a list of excluded file names (for example `.env`) that the agent is not allowed to summarise into memory. Neither is perfect. Both are needed.

### 3.7 Showing memory

The user must be able to see memory with a command (`/memory`). The view shows each file, its size in tokens, its last change, and the sections of agent notes. A user who cannot see what the model knows cannot correct it.

---

## 4. Gotchas

**4.1 Memory that grows without limit.** Notes are added and never removed, until the instruction costs more than the conversation. Enforce the size limit, and offer removal.

**4.2 A stale note that is treated as current.** "The tests are in `tests/`" becomes wrong when the folder is renamed. Store a date with each note, and show old notes with their age, so the user can judge them.

**4.3 A note that contradicts the project instructions.** If the user writes "use tabs" and an agent note says "use spaces", the model must not choose silently. Show the conflict, and let the user resolve it.

**4.4 Memory loaded from an untrusted location.** Only the project root's instruction file is loaded by default. A file pulled in from a dependency or a downloaded example must not become memory without a deliberate step.

**4.5 Memory that reveals the project to others.** A memory file is often committed to version control. Check, before a commit, that it holds nothing the user would not share. The `/memory` view should say whether the file is tracked by git.

**4.6 The model ignores memory.** A long memory file can be ignored, especially by a small model. Keep memory short and put the most important rule first. Test with a question that depends on the first line.

---

## 5. The lab in outline

You will:

1. write a project instruction file and load it into the system prompt,
2. measure its cost in the budget,
3. implement agent notes with an approval step,
4. enforce a size limit and a secret check,
5. build the `/memory` view,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Project instructions:** a user-written file, loaded into every session.
- **Agent note:** a fact the agent proposed, and the user approved, for future sessions.
- **Memory budget:** the size limit for memory, counted in the context budget.
- **Secret pattern:** a pattern that identifies text that looks like a key or token.
- **Section:** a heading in a memory file that groups notes of one kind.

---

## 7. Summary

- Memory is three kinds: user instructions, approved agent notes, and the conversation.
- Instructions are loaded into the system prompt, counted in the budget, and trusted.
- Agent notes are proposed and approved, never written silently.
- Memory has a size limit, a structure, and a secret check.
- The user can always see what the model knows, with `/memory`.

Continue to [exercises.md](exercises.md).
