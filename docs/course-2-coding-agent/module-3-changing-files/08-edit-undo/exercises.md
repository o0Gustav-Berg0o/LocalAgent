# Module 3, Lesson 8 — Exercises: Edit and Undo

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/FileChanges/`, the same slice as Lesson 7, with an `UndoCommand`. `SnapshotStore` goes in `Shared/Files/`, because cowork mode uses it in Lesson 14.
- Commit the sandbox project. Undo testing depends on a known starting point.
- Make sure Lesson 7's write tool works.
- Start the chat `llama-server`.

---

## Exercise 0 — Measure the cost of a full rewrite

**Goal:** know how many tokens a whole-file write costs, so you can see why edits exist.

**Do**

1. Pick a file of about 300 lines in the sandbox.
2. Ask the model to add one comment at the top of the file, using the Lesson 7 write tool. Let it complete.
3. Count the characters the model wrote in the tool call (you can estimate from the trace). Compare with the one line you actually changed.

**Observe**
- How many lines of the diff are real changes, and how many are reproduced text? Each reproduced line is an opportunity for an unwanted change.

**Checkpoint:** you have a count of real changed lines and reproduced lines, and a ratio.

---

## Exercise 1 — The exact-replacement function

**Goal:** a function that takes the file's text, the old text, and the new text, and returns the new text or a refusal. It checks uniqueness and normalises line endings.

**Do**

1. Create `Features/FileChanges/ExactReplace.cs`. The method takes the current text, the old text, and the new text, and returns a result: success with the new text, or a refusal with a reason and, where possible, the line numbers of the matches.
2. Normalise line endings in the file text and in the old text before matching. Remember the file's original line ending, and restore it when returning the text.
3. Count the matches. Zero matches: refuse with "not found". More than one match: refuse with the line numbers of each match, and "include more surrounding lines".
4. For zero matches, check whether a version of the old text matches when leading and trailing whitespace is ignored. If it does, say so in the refusal, with the line number. This helps with indentation mistakes (Gotcha 4.2).

Here is the outline. The matching is the part to write carefully:

```csharp
namespace LocalAgent.Features.FileChanges;

// Success: NewText is set. Refusal: Reason is set, and MatchLines lists 1-based line numbers
// of the matches (two or more), or of the whitespace-insensitive match (step 4).
public record ReplaceResult(bool Success, string? NewText, string? Reason, IReadOnlyList<int> MatchLines);

public static class ExactReplace
{
    public static ReplaceResult Apply(string fileText, string oldText, string newText)
    {
        // 1. Normalise line endings for matching only. Remember the original.
        // 2. Count occurrences of oldText. Use ordinal comparison, not culture-sensitive.
        // 3. Zero: refuse, and look for a whitespace-insensitive match to hint at.
        // 4. More than one: refuse, and list the line number of each match.
        // 5. Exactly one: replace it, restore the original line endings, return the text.
        throw new NotImplementedException();
    }
}
```

**Read this carefully.** Use an ordinal comparison. A culture-sensitive comparison can treat `ö` written two ways as equal, or differ between machines, and a match that depends on the machine is not a rule.

**Checkpoint:** run these inputs as tests:

| File text | Old text | Expected |
|---|---|---|
| `a\nb\nc` | `b` | replaced, one match |
| `a\nb\na` | `a` | refused, two matches with line numbers |
| `a\nb\nc` | `x` | refused, not found |
| `    return 1;` (4 spaces) | `return 1;` with no indent | refused, whitespace-insensitive match at the line |
| `a\r\nb` | `a\nb` | replaced, `\r\n` kept in the output |

---

## Exercise 2 — The edit tool

**Goal:** the model can change a file by exact replacement, through the same permission and apply path as Lesson 7.

**Do**

1. Create `Features/FileChanges/EditFileTool.cs` with inputs: the path, the old text, and the new text.
2. Resolve the path (Lesson 5). Read the file's current text.
3. Apply `ExactReplace` in memory. A refusal is returned to the model as a tool result, and nothing is proposed.
4. On success, create a proposal (Lesson 7) with the new text, and pass it to the review and apply path. The edit does not bypass the approval.

Start from this shell:

```csharp
using System.ComponentModel;

namespace LocalAgent.Features.FileChanges;

public sealed class EditFileTool(string root)
{
    [Description("Write the description yourself, as in Lesson 5: what it does, when to use it, when not to.")]
    public string EditFile(
        [Description("File path relative to the project root.")] string path,
        [Description("The exact text to replace. It must appear once in the file.")] string oldText,
        [Description("The text to put in its place.")] string newText)
    {
        // 1. Resolve the path (Lesson 5) and read the current text.
        // 2. ExactReplace.Apply. A refusal goes back to the model as the tool result.
        // 3. Success: WriteProposer.Propose with the new text, then the same review and apply
        //    path as write_file (Lesson 7). Return what happened, in one line.
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** An edit is a proposal in disguise. It must go through the same review as a whole-file write, and the diff shown to the user must be the diff of the replaced text. A student who lets an edit write directly has removed the safety of Lesson 7.

**Checkpoint:** ask the model to change one line of a sandbox file. The prompt shows a diff with that line only, and the file changes after approval.

---

## Exercise 3 — Several edits, one change

**Goal:** a list of edits to one file is applied in memory, in order, and written once.

**Do**

1. Extend `EditFileTool` so that its input can be a list of edits: each with an old text and a new text.
2. Apply each edit to the result of the previous one. Check uniqueness at each step, against the text as it is at that step.
3. If any edit is refused, refuse the whole set, and report which edit failed and why. Nothing is written.
4. If all edits succeed, create one proposal for the final text.

Add this to the edit code:

```csharp
public record EditStep(string OldText, string NewText);

// Applies the steps in order, each to the result of the one before. Stops at the first refusal,
// and says which step failed (1-based) and why. Nothing is written by this method.
public static (string? FinalText, string? Refusal) ApplyAll(string fileText, IReadOnlyList<EditStep> edits)
{
    throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** The second edit must be checked against the text after the first edit, not the original. An edit that adds a line can create a second match for a later edit's old text. Test that case on purpose.

**Checkpoint**
- Two edits to different parts of a file produce one diff and one approval.
- Two edits where the second fails leave the file unchanged, and the refusal names the second edit.
- An edit that creates a duplicate for a later edit is refused at the later step.

---

## Exercise 4 — Snapshot before every write

**Goal:** every change has a snapshot, and the snapshot is recorded before the change is applied.

**Do**

1. Create `Shared/Files/SnapshotStore.cs`. It writes snapshots to `.agent/undo/` in the project root, one folder per change, with a file for each changed file and a small manifest.
2. The manifest records: the change identifier, the time, each resolved path, the fingerprint before the change, and the fingerprint after the change.
3. Call `SnapshotStore` in the apply step, before the write. If the snapshot cannot be written, refuse the change (Gotcha 4.6).
4. After the write, record the fingerprint of the new content in the manifest.

Start from this shell:

```csharp
namespace LocalAgent.Shared.Files;

public record SnapshotFile(string ResolvedPath, string FingerprintBefore, string? FingerprintAfter);

// manifest.json in .agent/undo/<ChangeId>/, next to one copy of each file before the change.
public record SnapshotManifest(string ChangeId, DateTimeOffset Time, IReadOnlyList<SnapshotFile> Files);

public sealed class SnapshotStore(string projectRoot)
{
    // Before the write: copies each file and writes the manifest with FingerprintAfter = null.
    // Returns null if the snapshot could not be written; the caller then refuses the change.
    public SnapshotManifest? Record(IReadOnlyList<string> resolvedPaths)
    {
        throw new NotImplementedException(); // students write this
    }

    // After the write: fills in FingerprintAfter for every file in the manifest.
    public void Complete(SnapshotManifest manifest)
    {
        throw new NotImplementedException(); // students write this
    }

    // Used by /undo (Exercise 5) and by the history limit (Exercise 6).
    public SnapshotManifest? Latest() => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** Snapshot before writing, never after. A snapshot taken after the write holds the new content, and undo would restore nothing.

**Checkpoint:** after an approved edit, the `.agent/undo/` folder holds one folder with the old file and a manifest. The manifest's "after" fingerprint matches the new file on disk.

---

## Exercise 5 — `/undo` with the fingerprint check

**Goal:** `/undo` restores the last change, but only if the file is still in the state the change left it.

**Do**

1. Create an `UndoCommand` class (an `ISlashCommand`, Module 1, Lesson 1) in `Features/FileChanges/`, and register it in `Program.cs`. It reads the most recent manifest.
2. For each file in the manifest, compute the current fingerprint and compare it with the "after" fingerprint.
3. If every file matches, restore each snapshot, using `AtomicFile` from Lesson 3, and then remove the manifest from the undo history.
4. If any file does not match, refuse, and list the files that changed since the agent's change. Offer no restore for them, and say what the user can do: keep the new version, or restore by hand.

Add this to `SnapshotStore`, and call it from `UndoCommand`:

```csharp
// All or nothing. First checks every file's current fingerprint against FingerprintAfter.
// Any mismatch: restores nothing, and returns the paths that changed.
// All match: restores each file with AtomicFile (Lesson 3), then removes the manifest.
public (bool Restored, IReadOnlyList<string> ChangedSince) Undo(SnapshotManifest manifest)
{
    throw new NotImplementedException(); // students write this
}
```

And the command:

```csharp
namespace LocalAgent.Features.FileChanges;

public sealed class UndoCommand(SnapshotStore snapshots, int historyLimit) : ISlashCommand
{
    public string Name => "undo";
    public string Description => "Undo the agent's last change, if the files are as the change left them.";

    // 1. snapshots.Latest(). None: say there is nothing to undo.
    // 2. snapshots.Undo(manifest). Print what was restored, or the files that changed since.
    // 3. End with the history limit line from Exercise 6.
    public Task RunAsync(string arguments, CancellationToken cancellationToken)
        => throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** A partial undo leaves a project inconsistent: one file restored and another not. Check every file first, and restore only if all of them match. This is the all-or-nothing rule from Exercise 3, applied to undo.

**Checkpoint**
- After an approved edit, `/undo` restores the file to its content before the edit.
- After an approved edit, change the file in an editor, then `/undo`. It refuses, and lists the file.
- A change to two files is undone together, or refused together.

---

## Exercise 6 — Bound the history

**Goal:** the undo history has a limit, the limit is stated, and the oldest snapshot is removed when the limit is reached.

**Do**

1. Add a setting to the config: the maximum number of changes kept in the undo history. Use 50 as the default, and a range of 1 to 500.
2. When a new change is recorded and the limit is reached, remove the oldest manifest and its snapshot folder.
3. Show the limit in `/undo` output: "Undo keeps the last 50 changes. Older changes cannot be undone."

**Read this carefully.** Removing old snapshots is a deletion, and the user must be told it happens. Do not delete silently, and do not delete a snapshot that belongs to a change the user has not yet decided about.

**Checkpoint:** set the limit to 2, make three changes, and confirm that only two manifests remain, and that `/undo` says so.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Edit with no match | ask for an edit with wrong text | refused, not found, nothing changed |
| Edit with two matches | ask for an edit with a common line | refused, both line numbers shown |
| Indentation mismatch | ask for the same text without indentation | refused, the whitespace-insensitive match shown |
| Several edits, second fails | two edits, second with wrong text | nothing written |
| Undo after a second change | make two changes, then undo | the second change is undone, the first remains |
| Undo after a manual edit | change the file by hand after an agent change, then undo | refused, file listed |
| Two-file change, one file changed | change one of two files, then undo | refused for both |
| Snapshot folder skipped by search | search for text that is in a snapshot | no result from `.agent/undo` |
| Snapshot cannot be written | make `.agent` read-only | change refused before the write |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Compare the token cost in Exercise 0 with an edit of the same change. What would a model that rewrites files cost you over a whole project?
2. The undo refuses when a file changed after the agent's change. Is there a case where refusing is the wrong answer? What would you give the user instead?
3. A user asks for undo beyond the history limit. What do you say, and what could they do instead?

---

## Discuss

- An edit that matches exactly once is precise, but a model that is wrong about the text is refused. Which is more useful to the model: a precise refusal or a best guess? Why?
- Undo is a short history of the agent's changes, not version control. Where should a student's real history live, and why?
- The snapshot folder stores copies of the user's code. What should happen to it when the project is deleted or shared?

---

## Stretch

- Add a `/diff` command that shows the change that `/undo` would reverse, before the user confirms.
- Make the edit tool accept a line range as an alternative to exact text, and discuss the risk it adds.
- Store snapshots compressed. What does compression cost, and does it change the fingerprint rule?

---

## Time check

If you are past the planned time, finish Exercises 1, 2, and 5. The exact-replacement rule and the undo fingerprint check are the two parts that protect the user's work.
