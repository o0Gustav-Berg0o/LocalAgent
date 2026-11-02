# Lesson 8 — Exercises

## Exercise 1 — Write a labeled test set

**Goal:** fixed ground truth to measure against.

**Do**
1. Write 15 questions about your own notes, each with the filename(s) that should answer it.
2. Include a mix: some easy (exact-wording matches), some paraphrased, some that require combining two notes, and 2–3 that have **no** correct answer in your notes at all (expected result: nothing relevant).
3. Store this as a simple structured file (JSON or CSV): `question, expectedSources[]`.

## Exercise 2 — Build the recall@k harness

**Goal:** an automated, repeatable score.

**Do**
1. Write a command/script that, for each test question, calls your search function directly (bypassing the chat model entirely) at a chosen `k`.
2. For each question, check whether any of its `expectedSources` appears among the top-k results' source files.
3. Report per-question pass/fail and an overall recall@k percentage.
4. Handle the "no correct answer exists" questions correctly: for those, "passing" means nothing scored above your `MinScore` threshold, not that some filename matched.

## Exercise 3 — Use it to make a real decision

**Goal:** replace intuition with a measured comparison.

**Do**
1. Run the harness at your current settings and record recall@k.
2. Change exactly one thing — chunk size, `TopK`, whether the heading-path prefix is included, or whether the query instruction is applied — reindex, and run the harness again.
3. Record the new recall@k and the actual change.
4. Repeat for at least one more parameter change.

**Observe**
- Did your Lesson 3 intuition about chunk size match what the harness measured? Where did intuition and measurement disagree, if anywhere?

**Discuss**
- Why is a 15-question eval set a meaningful (if small) improvement over "I tried a few queries and it seemed fine," even though 15 is a tiny sample?
- recall@k says nothing about answer quality. What would a second metric, checking the final answer rather than retrieval, need to account for?
- How would you extend this harness as your notes collection grows to hundreds of files?
