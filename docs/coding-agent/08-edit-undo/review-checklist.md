# Module 3, Lesson 8 — Review Checklist

## Exact replacement

- [ ] The old text must occur exactly once. Zero and many matches are both refused, with useful messages.
- [ ] Matching uses an ordinal comparison, and line endings are normalised for matching only.
- [ ] The file's original line endings are kept in the output.

## Several edits

- [ ] Each edit is checked against the text after the previous edit.
- [ ] If any edit fails, nothing is written.

## Approval

- [ ] An edit goes through the same review and apply path as a whole-file write.

## Snapshots

- [ ] A snapshot is written before every change, and a change is refused if the snapshot fails.
- [ ] The manifest records the fingerprint before and after each change.
- [ ] The snapshot folder is skipped by search and by the file tools.

## Undo

- [ ] `/undo` checks every file's fingerprint before restoring any file.
- [ ] A file changed since the agent's change is listed, and not restored.
- [ ] A multi-file change is restored together or refused together.

## Bounded history

- [ ] The history limit is a setting, and `/undo` states it.
- [ ] Removing an old snapshot is announced, not silent.

## Reflection

- [ ] I measured the token difference between a rewrite and an edit for one change.
- [ ] I can explain why undo is not version control, and what I would tell a student to use instead.
