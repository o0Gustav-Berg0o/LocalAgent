# Module 2, Lesson 4 — Settings and the `/settings` Menu

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 2 and 3: streaming works, and every server value comes from the config file.
- Module 1, Lesson 1: you know the difference between setup and settings.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Build an interactive settings menu that changes one value at a time, and saves each change safely.
2. Define "effort" as a setting that maps to concrete limits, and document the mapping for the user.
3. Validate a new value before it is saved, and show the user why a value was refused.
4. Keep settings and state apart: a setting persists, a mode or a session value does not.
5. Make the menu usable with arrow keys, a clear current-value display, and a way out that never loses a change silently.
6. Respect the user's terminal: plain output when `NO_COLOR` is set, and no dependence on one terminal's features.

---

## 2. Why this lesson exists

Lesson 3 moved the values into a file. A file is good for a developer, but a user should be able to change a setting from inside the app without opening an editor. Two things make that harder than it looks:

- **Some values depend on each other.** A larger context size needs more memory. An effort level changes how long a model is allowed to think. A menu that lets the user pick any combination without warning produces settings that fail later, in a confusing way.
- **Changing a value is an action.** It writes a file, and it changes how the next answer is produced. The menu must show what will change, and must not save half of a change.

This lesson is about giving the user control over the settings, while keeping the file correct and the consequences visible.

---

## 3. Concepts

### 3.1 Settings, state, and modes

Three kinds of value are easy to mix up:

| Kind | Example | Where it lives | Survives a restart? |
|---|---|---|---|
| Setting | model name, endpoint, context size | `settings.json` | yes |
| Session state | the current conversation, the number of turns | memory, optionally a session file (Module 4) | only if saved as a session |
| Mode | code or cowork (Module 5) | per session, chosen by the user | not by default |

Only settings go in the settings menu. A mode is switched with its own command, because switching modes does not change the file. If the menu also changed the mode, the user would not know which change was permanent.

### 3.2 Effort, defined

"Effort" is a word many tools use, and no two tools define it the same way. For this course it is one setting with three levels, and each level maps to concrete limits that the user can read:

| Effort | Max answer length | Max tool calls per question | What it means for the user |
|---|---|---|---|
| low | 512 tokens | 2 | short, fast answers; one or two lookups |
| medium | 2048 tokens | 4 | the default; a short explanation plus a few lookups |
| high | 4096 tokens | 8 | longer answers and more searching; slower |

These numbers are starting points, written in the course, and they are constants in the code, not in the config. A user changes the level; the code decides what it means. Show the mapping in the menu, so the user knows what they are choosing. Exercise 3 asks you to test one of these numbers against a real question.

**Why not let effort change the model's "thinking" budget?** Some models support a reasoning budget. Not all local servers expose it, and the options differ between models. Keeping effort to length and tool limits works for any model, and the course can be extended later.

### 3.3 Validation before saving

A menu makes it easy to enter a bad value quickly. Validation must happen **before** the save, and the menu must say why a value was refused. For each setting, define:

- the allowed values or range,
- the message shown when a value is refused,
- any dependency on another setting, and the message when it is violated.

Example dependency: "context size must not exceed the server's context size". The menu refuses the value and shows the server value, so the user knows what to change.

### 3.4 Save one change at a time, safely

Each change is saved as soon as the user confirms it. Two rules:

1. **Write through a temporary file, then rename**, as in Module 1. A crash during a save cannot leave the file half-written.
2. **Keep the old value until the new one is on disk.** If the save fails, the menu shows the failure and keeps the old value in memory too. The screen must never show a value the file does not have.

### 3.5 Arrow-key navigation

A menu with arrow keys is more pleasant than a numbered list, but it needs care:

- The current value is shown next to each setting.
- The selected row is clearly marked. Use a marker character and, if colour is allowed, a colour. Never rely on colour alone, because `NO_COLOR` users and some terminals show no colour.
- Enter opens a setting for editing. Esc goes back, and never discards a change that was already confirmed.
- Quitting with unsaved changes asks the user what to do. Silent loss is not acceptable.

### 3.6 Terminal differences

The arrow keys, the colours, and the clearing of the screen behave differently across terminals. Build the menu so that it still works in a plain terminal, with numbers instead of arrows, if the key reading fails. The fallback is part of the feature, not a special case.

### 3.7 What a settings change does not do

A settings change does not:

- restart the server,
- download anything (Module 1 owns downloads),
- change the current answer that is already streaming,
- change the history of the current session.

It applies to the next request. Say so in the menu, next to each setting that needs it.

---

## 4. Gotchas

**4.1 Silent truncation.** A value that is too large may be accepted by the menu and cut down by the server. Validate against the server value, not only the app's own range.

**4.2 Effort and the tool limit interact.** A high effort with a low tool limit gives a confusing result: the model asks for more searches than it may make, and the answer ends early. The effort mapping must set both values together.

**4.3 A menu that changes state it does not show.** If the user changes the notes folder, does the index rebuild? If not, the menu must say "run `/reindex` to apply". Hidden consequences are the most common complaint about settings menus.

**4.4 Editing while the app is answering.** A menu opened during a streamed answer can change a value the answer is using. Either block the menu until the answer ends, or apply the change to the next request only, and say so.

**4.5 Numbers typed as text.** A value typed as `8k` or `8 192` must be parsed and checked, not stored as text.

**4.6 `NO_COLOR` and screen clearing.** Some users set `NO_COLOR` to get plain output. The menu must still work with it set: no colour codes, and the selected row still marked with a character.

---

## 5. The lab in outline

You will:

1. list every setting, its allowed values, its refusal message, and its dependency,
2. define the effort mapping and make the menu show it,
3. build the menu with validation before save and an atomic write,
4. add arrow-key navigation with a plain-number fallback,
5. handle quitting with unsaved changes,
6. test every dependency rule and every refusal message.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Setting:** a value that persists in the config file and that the user chooses.
- **Session state:** values that exist only while the app runs, or in a saved session.
- **Effort:** a user-facing level that maps to length and tool-call limits.
- **Dependency:** a rule that one setting's valid values depend on another.
- **Atomic write:** a save that either completes fully or leaves the old file untouched.
- **Fallback:** a simpler way to do the same thing when the preferred way is not available.

---

## 7. Summary

- Settings persist. State and modes do not.
- Effort is a named level with visible, documented limits.
- Every change is validated before it is saved, and every refusal says why.
- Saves are atomic, and the screen never shows a value the file does not hold.
- The menu works in a plain terminal and with `NO_COLOR` set.

Continue to [exercises.md](exercises.md).
