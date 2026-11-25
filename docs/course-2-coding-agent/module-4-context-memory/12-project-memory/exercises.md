# Module 4, Lesson 12 — Exercises: Project Memory

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/ProjectMemory/`, with a `MemoryCommand`. `SecretCheck` goes in `Shared/Security/`, because notes, skills, and session export use it.
- Start from a clean git state, with Lesson 11 working.
- Use the sandbox project. Create a folder for the test memory files inside it.
- Start the chat `llama-server`.

---

## Exercise 0 — Explain the same thing twice

**Goal:** feel the cost of having no memory, so the lesson has a reason.

**Do**

1. Start a new session. Tell the agent three things about the sandbox project: how to build it, where the tests are, and one style rule.
2. End the session. Start a new one. Ask the agent to build the project, and to add a test.
3. Note how many of the three things it knew without being told again.

**Observe**
- Which of the three was forgotten first? Why would that be?

**Checkpoint:** you have a count of remembered facts, and a sentence on what was forgotten.

---

## Exercise 1 — Write a project instruction file

**Goal:** a short, human-written instruction file in the project root, loaded at start.

**Do**

1. In the sandbox project root, create `AGENT.md`. Write at most twenty lines: the build command, the test command, the folder layout in three lines, and two style rules.
2. Create `Features/ProjectMemory/ProjectInstructions.cs` that finds the file in the project root and reads it. If the file is missing, it returns nothing, without an error.
3. Add the text to the system prompt, after the fixed instructions, under a heading that says where it came from ("Project instructions from AGENT.md").
4. Make the loader read the file once per session, not once per request.

Start from this shell:

```csharp
namespace LocalAgent.Features.ProjectMemory;

public static class ProjectInstructions
{
    public const string FileName = "AGENT.md";

    // Reads AGENT.md from the working folder once, at session start. A missing file gives null,
    // not an error. The caller keeps the text for the whole session.
    public static string? Load(string workingFolder)
    {
        throw new NotImplementedException(); // students write this
    }

    // The block added to the system prompt, after the fixed instructions:
    // a heading that says where the text came from, then the text.
    public static string ToPromptBlock(string text)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Read the file once, at the start of the session, and keep the text in memory. Reading it on every request costs disk time for no benefit, and makes the loaded text change in the middle of a conversation if the file is edited.

**Checkpoint:** start a new session and ask for the build command. The agent gives the command from `AGENT.md`, not a guess.

---

## Exercise 2 — Measure the cost

**Goal:** the instruction file's size in tokens, shown in the budget (Lesson 10).

**Do**

1. Add the instruction text as its own part of the context meter. The meter from Lesson 10 counted five parts; this makes it six, and the memory part is the sixth.
2. Show the memory size in tokens with the estimator from Lesson 10.
3. Write the size in `AGENT.md` as a comment at the top, once, so the user sees the cost where they edit the file. Keep it up to date by hand, or leave it out and show it only in `/memory`.

**Checkpoint:** the status or `/memory` shows a memory size, and it matches the estimate for the file.

---

## Exercise 3 — Propose an agent note

**Goal:** the agent can propose a note for memory, and the user decides.

**Do**

1. Add a tool `propose_memory_note` with inputs: a section name (one of: commands, layout, conventions, known issues), the note text, and a reason.
2. The tool does not write anything. It returns a proposal, and the ask prompt (Lesson 6) shows the section, the text, and the reason, with the choices yes, no, and edit.
3. On yes, append the note to the agent's memory file, under the section, with the date. On edit, let the user change the text first. On no, nothing is written.
4. Store agent notes in `.agent/memory.md`, separate from `AGENT.md`, so the user's file is never changed by the agent.

Here is the note format. Keep it readable as plain Markdown:

```markdown
## Layout
- (2026-11-25) The config loader is in Config/Loader.cs. Reason: user asked where settings are read.
```

Start from these shells:

```csharp
using System.ComponentModel;

namespace LocalAgent.Features.ProjectMemory;

public record MemoryNote(string Section, string Text, string Reason, DateOnly Date);

public sealed class MemoryNotes(string notesPath)   // .agent/memory.md
{
    public static readonly string[] Sections = ["commands", "layout", "conventions", "known issues"];

    // Appends under the section heading, with the date (the format above).
    public void Append(MemoryNote note) => throw new NotImplementedException(); // students write this

    // Section to notes, numbered from 1 in each section, as /memory shows them (Exercise 6).
    public IReadOnlyDictionary<string, IReadOnlyList<MemoryNote>> ReadAll()
        => throw new NotImplementedException(); // students write this

    public bool Remove(string section, int number) => throw new NotImplementedException(); // students write this
}

public sealed class ProposeMemoryNoteTool(MemoryNotes notes)
{
    [Description("Write the description yourself: when a fact is worth remembering, and when not.")]
    public string ProposeMemoryNote(
        [Description("One of: commands, layout, conventions, known issues.")] string section,
        [Description("The note, one or two sentences.")] string note,
        [Description("Why this is worth remembering.")] string reason)
    {
        // 1. Refuse an unknown section, and list the valid ones.
        // 2. The size and secret checks (Exercise 4).
        // 3. Ask: yes, no, or edit (AskPrompt from Lesson 6, with an edit choice).
        // 4. Yes or edited: notes.Append. No: write nothing. Return what happened.
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The agent note file is separate from the user's instruction file. The user's file is never written by a tool. A student who merges the two has removed the user's control over their own instructions.

**Checkpoint**
- Ask the agent to remember a fact. The prompt shows the proposal, and nothing is written until yes.
- On yes, `.agent/memory.md` has the note under the right section, with a date.

---

## Exercise 4 — Size limit and secret check

**Goal:** memory cannot grow without limit, and cannot hold a key.

**Do**

1. Add a size limit for agent memory to the config, default 2000 tokens by the estimator. Before approving a note, check that the note fits. If it does not, refuse with the current size and the limit, and suggest removing or merging notes.
2. Add a secret check. Refuse a proposed note that contains a pattern that looks like a key: a long run of letters and digits with a common prefix such as `sk-`, `ghp_`, or a line with `password=`. Write the patterns in the config, so they can be changed.
3. Make the refusal say what was matched, without printing the secret itself. Show the first four characters and the length.

Start from this shell. Session export (Lesson 13, stretch) reuses it:

```csharp
using System.Text.RegularExpressions;

namespace LocalAgent.Shared.Security;

public static class SecretCheck
{
    // Patterns come from the config (step 2). Returns null when nothing matched, or a message
    // with the pattern name, the first four characters of the match, and its length.
    // Never returns the full match.
    public static string? Find(string text, IReadOnlyList<Regex> patterns)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** A secret check is a heuristic. It catches the common formats and misses others. Say so in the README of your app, and do not describe it as a guarantee.

**Checkpoint**
- A note that would exceed the limit is refused with the size message.
- A note containing `sk-` followed by a long string is refused, and the message does not print the full key.

---

## Exercise 5 — The `/memory` view

**Goal:** the user can see every memory source, its size, its age, and whether git tracks it.

**Do**

1. Add a `MemoryCommand` (an `ISlashCommand`) in `Features/ProjectMemory/`, and register it in `Program.cs`. It prints one block per file: the path, the size in tokens, the last change date, and for agent notes, the sections with the number of notes in each.
2. Check whether each file is tracked by git, by asking git for the file's status. If git is not available, say so, and do not fail.
3. Warn when `.agent/memory.md` is tracked and contains notes, because a committed memory file shares its contents with everyone who has the repository.

Start from this shell. Run git through the `CommandRunner` from Lesson 9, so it gets the same timeout and argument list:

```csharp
namespace LocalAgent.Features.ProjectMemory;

public enum TrackedState { Tracked, NotTracked, GitUnavailable }

public static class GitStatus
{
    // Asks git whether the file is tracked, for example with "git ls-files --error-unmatch <path>".
    // Read-only: never adds, never commits. Git missing or failing gives GitUnavailable.
    public static Task<TrackedState> IsTrackedAsync(CommandRunner runner, string workingFolder, string path)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

And the command:

```csharp
namespace LocalAgent.Features.ProjectMemory;

public sealed class MemoryCommand(string workingFolder, MemoryNotes notes, CommandRunner runner) : ISlashCommand
{
    public string Name => "memory";
    public string Description => "Show the memory files: size, age, sections, and whether git tracks them.";

    // No arguments: one block per file (step 1), with the git state (step 2) and the warning (step 3).
    // "remove <section> <number>": Exercise 6.
    public Task RunAsync(string arguments, CancellationToken cancellationToken)
        => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** The git check must not change anything. It asks for status, and never adds or commits. A check that stages a file is a write, and needs approval (Lesson 7).

**Checkpoint**
- `/memory` shows both files, with sizes and dates.
- With the memory file tracked and non-empty, the warning appears.

---

## Exercise 6 — Remove and merge notes

**Goal:** the user can remove a note, so memory stays current.

**Do**

1. Add `/memory remove <section> <number>`. Each note in a section has a number shown by `/memory`. Removing asks for confirmation, then deletes that note only.
2. Show the note's age in `/memory`. Mark any note older than 90 days, and say it may be stale.

**Checkpoint:** remove one note by number, and confirm that only that note is gone. Run `/memory` again to see the new count.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| No `AGENT.md` | rename the file | session starts normally, no memory section |
| Very long `AGENT.md` | paste 500 lines | warning with its size, loaded anyway or refused by the limit |
| Note proposed, answered edit | answer edit and change the text | the edited text is saved |
| Note proposed, answered no | answer no | nothing written |
| Secret in a proposed note | include a token-like string | refused, the secret not printed |
| Note in wrong section name | the agent proposes an unknown section | refused, valid sections listed |
| Memory conflicts with instructions | `AGENT.md` says tabs, a note says spaces | the conflict is shown, the user decides |
| Git not installed | run `/memory` where git is unavailable | the view works, the git check says unavailable |
| Memory file tracked and non-empty | commit the memory file, then run `/memory` | warning shown |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which fact did the agent remember best after you added memory? Which did it still get wrong, and why?
2. The user's instruction file and the agent's notes look alike. What is the difference in trust, and how does the model's behaviour show it?
3. A note is approved and later becomes wrong. What should the user do, and how does the `/memory` view help them?

---

## Discuss

- Memory costs tokens on every request. Is there a point at which the agent should use a tool to look something up instead of remembering it? How would you decide?
- An agent note is a fact the model believes. Who is responsible when a note is wrong, and what would you tell a user who blames the agent?
- The secret check is a heuristic. What does it teach a student about the difference between a rule that is checked and a rule that is guaranteed?

---

## Stretch

- Add a note expiry setting, so that notes older than a chosen age are shown but not loaded.
- Add a rule that a note must cite a file, and check that the file exists when the note is loaded.
- Support a second instruction file for a subfolder, loaded only when the agent works in that folder. What does this add to the budget?

---

## Time check

If you are past the planned time, finish Exercises 1, 3, and 4. The loaded instructions, the approved notes, and the secret check are the three parts that make memory safe to use.
