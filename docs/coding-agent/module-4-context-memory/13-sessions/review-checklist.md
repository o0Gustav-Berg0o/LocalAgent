# Module 4, Lesson 13 — Review Checklist

## Format

- [ ] The session file has a header with a format name, a version, and the session identifier.
- [ ] Each event is one JSON line, with a version, a sequence number, a time, and a kind.
- [ ] `FORMAT.md` explains every field, so a student can read a session without the app.

## Writing

- [ ] Events are appended, one per line, and flushed.
- [ ] A session has one writer, enforced by a lock check on open.
- [ ] Large content is referenced by size and fingerprint, not copied.
- [ ] No secrets appear in any event.

## Reading and resume

- [ ] A damaged line is skipped, counted, and reported with its number.
- [ ] A newer format version is refused with both versions named.
- [ ] Resume rebuilds the history in order, and checks that every call has a result.
- [ ] A missing result is replaced by a "not recorded" result, and the user is told.
- [ ] Resume never runs a tool again.

## Picker and delete

- [ ] The picker sorts by the time of the last event, and lists only sessions with a user message.
- [ ] Delete shows the session's details and needs the word `yes`.

## Compaction

- [ ] A compaction writes an event with its summary and the last sequence number it covers.
- [ ] Resume starts from the latest compaction, and the earlier events remain in the file.

## Reflection

- [ ] I killed the app mid-tool-call and resumed the session successfully.
- [ ] I can say which content a session must never hold, and why.
