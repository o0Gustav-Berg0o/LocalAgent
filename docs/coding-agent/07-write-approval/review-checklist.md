# Module 3, Lesson 7 — Review Checklist

## Structure

- [ ] Propose, review, and apply are three functions, and the tool calls them in that order.
- [ ] `Propose` writes nothing to disk.

## Diff

- [ ] The diff is computed from the file on disk, split into lines.
- [ ] A line-ending change alone shows no changed lines.
- [ ] Long diffs are cut at a stated number of lines, with the count.

## Fingerprint

- [ ] The proposal stores the fingerprint of the file's bytes, or of "absent" for a new file.
- [ ] Apply recomputes the fingerprint immediately before the write and refuses on a mismatch.

## Atomic write

- [ ] The temporary file is in the same folder as the target.
- [ ] Size and fingerprint are checked before the move, and again after it.
- [ ] A failure before the move leaves the original untouched and removes the temporary file.

## Limits

- [ ] Extension, size, and folder limits are checked before any prompt is shown.
- [ ] Each refusal names the limit that was broken.
- [ ] Missing folders are refused, not created.

## Reflection

- [ ] I simulated a file change during review and saw the apply refuse.
- [ ] I can name the failure I consider worst, and what prevents it.
