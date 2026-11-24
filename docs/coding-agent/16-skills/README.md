# Module 6, Lesson 16 — Skills: Instructions That Load When Needed

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 4, Lesson 10: the budget counts everything in the request.
- Module 4, Lesson 12: project memory and why it costs tokens on every request.
- Module 2, Lesson 5: tool descriptions as prompts.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why a large set of instructions should not sit in the system prompt, and how lazy loading keeps the prompt small.
2. Define a skill as a folder with a short index entry and a full body, and write the index entry so the model can choose correctly.
3. Build a tool that loads a skill's full body on demand, and count the cost of the index and the body separately.
4. Treat a skill's body as instructions from the user's own files, and decide what it may and may not change.
5. Limit a skill's size, its number, and the tools it may ask for.
6. Test whether the model loads the right skill, and measure it.

---

## 2. Why this lesson exists

A coding agent accumulates know-how: how to write a migration, how to structure a test, how to release a package, how to review a change. Each of these is a few hundred lines of instruction. Putting all of them in the system prompt costs thousands of tokens on every request, whether or not the task needs them. The budget (Lesson 10) fills up before the work starts, and the model gets worse at following the instructions that matter most, because they are buried among the ones that do not.

Skills solve this by splitting each instruction set in two. The **index** is short: a name and a description, always loaded, so the model knows the skill exists. The **body** is long: the full instructions, loaded only when the model asks for the skill. The model pays for the body only when it needs it.

---

## 3. Concepts

### 3.1 The skill as a folder

A skill is a folder with one file, `SKILL.md`, at its root. The file starts with a short header (the index entry) and continues with the body:

```markdown
---
name: write-migration
description: Use when the task asks for a database schema change. Explains how migrations are named and tested in this project.
---

# Writing a migration

1. Name the file with the date and a short verb: 2026-11-25-add-user-email.sql.
2. ...
```

The header is the index. The body is everything after the second line of dashes. The name must be lowercase with hyphens, and the description must say when to use the skill, not only what it is.

### 3.2 The index is a prompt

The index is sent with every request, so it is a prompt, and the model chooses skills by reading it. A description that says "Migrations" is too vague: the model cannot tell when it applies. A description that says "Use when the task changes a database schema" tells the model when to load the skill. Write descriptions for the choice, not for the reader of the folder.

The index for all skills together should stay under a small budget, for example 500 tokens. Ten skills with long descriptions can cost more than the instructions they hide.

### 3.3 Loading on demand

The model loads a skill by calling a tool, for example `load_skill` with the name. The tool returns the body, and the body becomes part of the history like any tool result. The body costs tokens only from that point, and compaction (Lesson 11) treats it as a tool result, so it can be placeholdered when it is no longer needed.

Two rules follow:

- the tool accepts only names from the index. A name that is not in the index is refused, and the refusal lists the valid names.
- the body is loaded once per session, and a second request for the same skill returns a short note that it is already loaded.

### 3.4 A skill is the user's file, and still data

A skill body is text in a file the user chose to install. The model follows it. But a skill can also contain text copied from the internet, or written by someone else, so the body is treated as instructions with the authority of the user's own files, and no more.

A skill may say how to do a task. It may not say that the permission rules do not apply. The rules are code, and a skill cannot change code. If a skill asks for a tool, the request goes through the same permission check as any other request.

### 3.5 Skills and tools

A skill can name tools the body expects to use. The name is a hint, not a grant. If the skill says "use the shell to run the migration tests", the shell tool is still gated by the allowlist (Lesson 9). A skill that needs a tool the user has not allowed must say so, and the agent must ask.

### 3.6 Where skills live

Skills live in a skills folder in the user's application data, and optionally in a `skills` folder in the project. A project skill overrides a user skill with the same name, and the app says so when it loads the index. Overrides are a common source of confusion, so make them visible.

### 3.7 Limits

Skills need limits, like everything else in the context:

- a maximum size for a body, for example 8 KB,
- a maximum number of skills in the index, for example 30,
- a maximum size for the index as a whole, for example 500 tokens by the estimator.

When a limit is reached, the app refuses the new skill and says which limit it hit, rather than silently dropping it.

---

## 4. Gotchas

**4.1 Descriptions that never match.** A description the model does not recognise is a skill that is never loaded. Test each skill with a task it should match.

**4.2 Descriptions that match everything.** A description such as "Use for any coding task" loads on every request and costs the body each time. Keep descriptions specific.

**4.3 A body that refers to a file that is not there.** A skill that says "see the template in templates/x.md" needs that file too. Either bundle the file in the skill folder, and allow the loader to read it, or say in the body that it is missing.

**4.4 Two skills that conflict.** One skill says to use tabs, another says to use spaces. The model must not choose silently. The index can carry a warning when two descriptions overlap, and the body can say what it takes precedence over.

**4.5 Skills that grow.** A body that has been extended for a year is hard to read, and expensive to load. Review skills on a schedule, and keep each one to one task.

**4.6 A skill loaded for the wrong task.** The model loads a skill for a similar but different task, and follows it. Ask the model, in the answer, to say which skill it used and why, so the user can see the choice.

---

## 5. The lab in outline

You will:

1. write a skill folder with an index header and a body,
2. build the index and measure its cost,
3. build the `load_skill` tool with the name check and the once-per-session rule,
4. test whether the model chooses the right skill, using a set of tasks,
5. add the limits and the project override,
6. test the failure cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Skill:** a folder with an index entry and a body, loaded on demand.
- **Index entry:** the short header that the model sees on every request.
- **Body:** the full instructions, loaded only when needed.
- **Lazy loading:** loading a part only when it is needed.
- **Override:** a project skill that replaces a user skill with the same name.
- **Hint:** a named tool in a skill that does not grant permission to use it.

---

## 7. Summary

- Skills split instructions into a short index, always loaded, and a long body, loaded on demand.
- The index is a prompt, so its descriptions say when to use each skill.
- A body is the user's file and still data; it cannot change the rules.
- Skills have size, count, and index limits, and overrides are shown.
- Test that the model picks the right skill, and ask it to say which one it used.

Continue to [exercises.md](exercises.md).
