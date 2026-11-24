# Module 4, Lesson 11 — Review Checklist

## Order of operations

- [ ] Placeholdering runs before compaction, and compaction runs only if the budget is still above the act threshold.
- [ ] The full transcript is written to disk before a compaction, and the write is checked.
- [ ] A failed compaction leaves the history unchanged.

## Summary

- [ ] The summary uses the fixed headings: goal, decisions, files, open items, constraints.
- [ ] The instruction text is short, and each rule has a reason.
- [ ] Every file name in a summary appears in the transcript, checked by a test or a documented limit.

## Verbatim

- [ ] The last four turns, or the configured number, are kept exactly as they were.
- [ ] No turn is split, and no tool call is separated from its result.
- [ ] Pinned rules appear in the summary word for word.

## Commands and settings

- [ ] `/compact` reports what it did, and says "nothing to compact" when there is nothing to do.
- [ ] Automatic compaction is off by default, and prints a line when it runs.

## Fidelity

- [ ] I wrote ten facts before the conversation, and asked ten questions that only the facts can answer.
- [ ] I recorded the fidelity as a number, and listed the facts that were lost.

## Reflection

- [ ] I can say which kind of fact was lost most often, and why.
- [ ] I would not trust a summary without a check, and I can say what check I would use.
