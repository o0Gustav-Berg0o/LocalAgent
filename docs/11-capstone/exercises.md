# Lesson 11 — Exercises

## Exercise 1 — The full demo run

**Goal:** prove the whole system works together, not just lesson-by-lesson in isolation.

**Do**
1. Start from a freshly reindexed notes folder: delete `index.json` from your project root (or run `/reindex full` if you built that command) so nothing is loaded from a stale cache, then `dotnet run`.
2. Run through, in one session, in this order:
   - A question clearly answerable from your notes (check: searched, grounded, cited).
   - A question with no answer in your notes (check: explicit "I don't know," not a guess).
   - A question containing an exact identifier/error code (check: hybrid search finds it).
   - A request to create a new note (check: confirmation shown with exact content, confirm it, verify the file and index update).
   - A request to create a note that you then **decline** (check: nothing written).
   - A plain conversational question with no tool need (check: no spurious tool call).
3. Record the full trace output for this run — this is your evidence, not just a claim that it works.

## Exercise 2 — Written retrospective

**Goal:** reflect critically, not just report completion.

**Do, in writing:**
1. Which lesson's concept surprised you the most — something you expected to be simple that turned out subtle, or vice versa?
2. Where is your current build still weakest? Be specific (e.g. "chunking still fragments tables," not "retrieval could be better").
3. If a non-technical note contained a particularly clever prompt injection, do you believe your current defenses (Lesson 7) would catch it? Why or why not?
4. Pick one extension point from the course's `README.md` "Credits"/extension list (reranking, a real vector database, OpenTelemetry, summarization-based history) and write 3–5 sentences on how you'd actually implement it, given what you now know about this codebase.

## Exercise 3 — Teach it

**Goal:** the fastest way to find gaps in your own understanding is to explain it to someone else.

**Do**
1. Pick one lesson (not Lesson 1 — pick something with more nuance, like Lesson 4, 7, or 9) and explain it out loud, in your own words, to another person or by recording yourself, without looking at the lesson's `README.md`.
2. Afterward, compare what you said against the lesson content. What did you leave out or get slightly wrong?
