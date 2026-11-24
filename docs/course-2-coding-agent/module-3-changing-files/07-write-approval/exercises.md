# Module 3, Lesson 7 — Exercises: Write with Approval

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state. Commit the sandbox project first, so you can restore it after each test.
- Keep the sandbox project separate from your real work.
- Start the chat `llama-server`.

---

## Exercise 0 — Find out how the model writes today

**Goal:** know whether anything can write now, before you add the write tool.

**Do**

1. Search the app for any call that writes a file (`File.Write`, `WriteAllText`, `Move`, and similar).
2. For each one, write whether it is behind the permission check, and whether it writes an existing file or creates a new one.
3. The note-creation tool from the RAG course is one of them. Note how it asks for approval.

**Observe**
- The RAG course note tool approves a whole new file with a title. What does that approval not tell the user, that a change to an existing file would?

**Checkpoint:** you have the list, with a yes or no for "behind the check" on each entry.

---

## Exercise 1 — Split propose, review, apply

**Goal:** three separate functions, so each can be tested alone.

**Do**

1. Create `Tools/Write/WriteProposal.cs`, a record with: the resolved path, the current content (or null if the file does not exist), the proposed content, and the fingerprint of the current content.
2. Create `Tools/Write/WriteProposer.cs` with a method that takes a path and the proposed content, reads the current file, computes the fingerprint, and returns a `WriteProposal`. It writes nothing.
3. Create `Tools/Write/WriteApplier.cs` with a method that takes a `WriteProposal` and writes it. It does not ask anyone anything. The approval happens before it is called.
4. Make the `write_file` tool call the proposer, then the permission check, then the applier, in that order, and never skip one.

Here is the shape of the separation:

```csharp
// Propose: pure with respect to the disk. Reads, computes, returns. Never writes.
WriteProposal proposal = WriteProposer.Propose(resolvedPath, proposedText);

// Review: the permission check and the prompt. Returns the user's answer.
bool approved = await Permission.ReviewAsync(proposal);

// Apply: only if approved. Checks the fingerprint, then writes atomically.
if (approved) WriteApplier.Apply(proposal);
```

**Read this carefully.** The applier must not trust the proposal blindly. It recomputes the fingerprint and compares it with the one stored in the proposal (Exercise 4). The separation is what makes that check possible.

**Checkpoint:** calling `Propose` on a file and then checking the disk shows no change. Calling `Apply` on a proposal with a wrong fingerprint refuses, and the file is untouched.

---

## Exercise 2 — Compute and show a diff

**Goal:** a readable diff of the current file and the proposed content, computed from the disk.

**Do**

1. Create `Tools/Write/DiffRenderer.cs`. It takes the old lines and the new lines, and returns the diff as text, with context lines and the three markers `  ` (context), `- ` (removed), `+ ` (added).
2. Use a line-based diff. A simple approach is the longest common subsequence of lines, which is easy to test on small inputs. If the input is large, use a library, and note the choice.
3. Limit the diff output to 120 lines. If the diff is longer, show the first 120 lines and say how many more there are.
4. Keep the file's line endings in the comparison. Split on `\n` and trim a trailing `\r`, and do not treat a changed line ending as a change to the text.

**Read this carefully.** Students often compare whole strings and get "the whole file changed". Split into lines first. A diff is useful only when it shows a few lines out of many.

**Checkpoint**
- Changing one line in a 50-line file shows that line as removed and added, with two context lines on each side.
- Converting a file from `\r\n` to `\n` without changing text shows no changed lines.
- A diff longer than 120 lines is cut, with the count.

---

## Exercise 3 — Fingerprint the file at proposal time

**Goal:** the proposal records exactly which version of the file it was computed against.

**Do**

1. In `WriteProposer.Propose`, compute the SHA-256 of the file's bytes, or of `null` if the file does not exist, and store it in the proposal.
2. Write the function so that it reads the bytes, not the text. A text read can change the bytes through encoding handling, and the fingerprint must match the bytes on disk.
3. Use the same hashing function the download in Module 1 used, so the code is shared.

**Checkpoint:** two proposals of an unchanged file have the same fingerprint. Change one character in the file, and a new proposal has a different fingerprint.

---

## Exercise 4 — Refuse a changed file at apply time

**Goal:** if the file changed between the proposal and the approval, the apply refuses and asks for a new review.

**Do**

1. In `WriteApplier.Apply`, recompute the fingerprint of the file on disk, right before writing.
2. Compare it with the fingerprint in the proposal. If they differ, do not write. Return a clear result: "the file changed after the change was shown. Review it again."
3. For a new file (the proposal's fingerprint is for `null`), refuse if the file now exists, with the same kind of message.

**Read this carefully.** The check must happen after the approval and immediately before the write. A check at proposal time does not help, because the file can change in the gap. Put the check next to the write, not in the caller.

**Checkpoint:** create a proposal, open the file in an editor and change one line, then approve the proposal. The apply refuses, and the file keeps the editor's change.

---

## Exercise 5 — Write atomically and verify

**Goal:** the write either completes or leaves the original untouched, and the result is checked.

**Do**

1. In `WriteApplier.Apply`, write the proposed content to a temporary file in the same folder (for example `name.txt.tmp-<guid>`).
2. Close the temporary file, check its size against the expected size, and recompute its fingerprint and compare it with the fingerprint of the proposed content.
3. Move the temporary file over the original, in one step.
4. After the move, read the file back and check its fingerprint again. If it differs, report the error and say what state the file is in.
5. If any step before the move fails, delete the temporary file and leave the original untouched.

**Read this carefully.** The move is the only step that changes the original, and it is the one that must be atomic. Write the temporary file in the same folder, not in `%TEMP%`, so the move stays on one volume.

**Checkpoint**
- A normal approved write changes the file, and no temporary file is left behind.
- Simulate a failure before the move (for example, make the folder read-only after the temporary file is written). The original is unchanged, and the temporary file is removed.

---

## Exercise 6 — Folder, type, and size limits

**Goal:** a write can only touch what it is allowed to touch, and larger changes are refused with a clear message.

**Do**

1. Add three settings to the config: the allowed extensions, the maximum size in bytes for one write, and the folders that may be written (default: the project root).
2. In the proposer, check each limit before the review. A refused proposal never reaches the user as a prompt, and returns a message to the model that explains the limit.
3. Make the message name the limit: "extension .exe is not in the allowed list", "the change is 300 KB; the limit is 200 KB. Split the change into smaller files."

**Checkpoint:** a write to a `.exe` file is refused before any prompt. A write of a file above the size limit is refused with the size message. A write inside an allowed folder with an allowed extension reaches the prompt.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Change one line | edit one line through the tool | diff shows one removed, one added, context |
| Line-ending change only | convert `\r\n` to `\n`, no text change | no changed lines shown |
| File changed during review | edit the file in an editor between proposal and approval | apply refuses, file keeps the editor's change |
| New file, path in missing folder | propose a file in a folder that does not exist | refused, folder not created |
| New file that now exists | create the file after the proposal, then approve | apply refuses |
| Encoding with Swedish letters | change a line containing `ö` in a UTF-8 file | letters preserved after the write |
| Very large change | propose a change above the size limit | refused before the prompt |
| Denied at the prompt | answer no | file unchanged, model told the change was refused |
| Write with no permission to the folder | make the folder read-only | clear error before any change |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which failure would a user notice first: a wrong diff, a changed file that was overwritten, or a missing folder that was created? Which is the worst?
2. The fingerprint check refuses when the file changed. Is there a case where refusing is wrong? What would you change?
3. Compare the note-creation approval from the RAG course with this approval. What does the user now see that they did not see before?

---

## Discuss

- The user approves what they see, not what the model meant. How close can the diff come to the model's intention, and what should happen when they differ?
- A write is only as safe as the review. What can a user miss in a diff, and how could the tool help them see it?
- Why does the apply step recheck the fingerprint, when the proposal already checked it? Who else could change the file in between?

---

## Stretch

- Add a "show full diff" option that opens the diff in a file the user can scroll.
- Add a check that the proposed file still parses, for example that a `.json` file is valid JSON, before the review. What does it prevent, and what does it not prevent?
- Keep the previous version of every write in a hidden folder, as the first step toward undo in Lesson 8. What must be excluded from that folder?

---

## Time check

If you are past the planned time, finish Exercises 1, 3, 4, and 5. The fingerprint check and the atomic write are the two protections that matter most in this lesson.
