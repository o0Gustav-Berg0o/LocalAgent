# Lesson 8 — Exercises

Both `llama-server` windows must still be running. All paths are relative to your project root.

## Exercise 1 — Write a labeled test set

**Goal:** fixed ground truth to measure against.

**Do**

- [ ] **Create `eval/questions.json`** (create the `eval` folder). Write 15 entries, each with a question and the filename(s) that should answer it:
   ```json
   [
     { "question": "When should I plant tomatoes?", "expectedSources": ["garden-notes.md"] },
     { "question": "What's the ideal soil pH for tomatoes?", "expectedSources": ["garden-notes.md"] },
     { "question": "How do I configure Rust's borrow checker?", "expectedSources": [] }
   ]
   ```
- [ ] Include a mix: some easy (near-exact wording from a note), some paraphrased, some that need two notes combined, and 2–3 with an empty `expectedSources` array — topics genuinely absent from your notes.

## Exercise 2 — Build the recall@k harness

**Goal:** an automated, repeatable score, with no chat model involved.

**Do**

- [ ] **Add an `/eval` command** that reads `eval/questions.json`, runs each question through your embedding + `VectorStore.TopK` path directly (the same code `search_notes` uses — not through the agent loop), and checks whether any `expectedSources` filename appears among the top-`k` chunk sources. Placement: with the command helper from Lesson 3's checkpoint, this is one new `case "/eval":` in `CommandHelper.HandleAsync` (`case "/eval": await EvalAsync(ctx); break;`) plus a new private `EvalAsync` method in the same file — no change to the `while` loop. Put the `record EvalQuestion(...)` at the bottom of `Helpers/CommandHelper.cs`, outside the `CommandHelper` class (a `record` can't be declared inside a method body):
   ```csharp
   private static async Task EvalAsync(CommandContext ctx)
   {
       var json = await File.ReadAllTextAsync("eval/questions.json");
       var questions = System.Text.Json.JsonSerializer.Deserialize<List<EvalQuestion>>(json)!;
       int passed = 0;

       foreach (var q in questions)
       {
           var queryVector = (await ctx.Embeddings.GenerateVectorAsync($"Instruct: Given a question, retrieve relevant passages\nQuery:{q.Question}")).ToArray();
           var hits = ctx.Store.TopK(queryVector, k: 6).Where(h => h.Score >= 0.3f).ToList();
           var foundSources = hits.Select(h => h.Chunk.SourceFile).ToHashSet();

           // A question with an empty expectedSources is a "should find nothing" case -- it
           // passes only if MinScore filtered out every hit; otherwise it passes if any
           // expected file showed up among the results.
           bool ok = q.ExpectedSources.Count == 0
               ? hits.Count == 0
               : q.ExpectedSources.Any(foundSources.Contains);

           Console.WriteLine($"{(ok ? "PASS" : "FAIL")}  {q.Question}");
           if (ok) passed++;
       }
       Console.WriteLine($"\nrecall@6: {passed}/{questions.Count} = {100.0 * passed / questions.Count:F0}%");
   }

   // Bottom of Helpers/CommandHelper.cs, outside the CommandHelper class:
   public record EvalQuestion(string Question, List<string> ExpectedSources);
   ```
- [ ] `dotnet run`, type `/eval`, and check the per-question PASS/FAIL output against what you expect by eye.

## Exercise 3 — Use it to make a real decision

**Goal:** replace intuition with a measured comparison.

**Do**

- [ ] Run `/eval` at your current settings and write down the recall@6 percentage.
- [ ] **Change exactly one thing** — e.g. set `MaxChunkChars` to `500` in your `MarkdownChunker.Chunk` call — `/reindex`, then run `/eval` again.
- [ ] Write down the new recall@6 and the actual delta (not "it seemed better" — the number).
- [ ] Repeat for at least one more change: try `k: 4` instead of `6` in `/eval`'s `TopK` call, or temporarily strip the heading-path prefix from indexing like you did in Lesson 3's Exercise 2, `/reindex`, and re-run `/eval`.

**Observe**
- Did your Lesson 3 intuition about chunk size match what the harness actually measured? Where did intuition and measurement disagree, if anywhere?

**Discuss**
- Why is a 15-question eval set a meaningful (if small) improvement over "I tried a few queries and it seemed fine," even though 15 is a tiny sample?
- recall@k says nothing about answer quality. What would a second metric, checking the final answer rather than retrieval, need to account for?
- How would you extend this harness as your notes collection grows to hundreds of files?
