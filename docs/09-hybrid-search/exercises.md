# Lesson 9 — Exercises

Both `llama-server` windows must still be running. All paths are relative to your project root.

## Exercise 1 — Prove the failure mode first

**Goal:** before building anything, confirm the problem you're solving actually exists in your data.

**Do**

- [ ] **Add a note with a specific identifier**, e.g. `samples/notes/esp32-wifi.md`:
   ```markdown
   # ESP32 WiFi

   When the device loses its connection, the system fires `WIFI_EVENT_STA_DISCONNECTED`.
   Handle it by calling `esp_wifi_connect()` again after a short backoff.
   ```
- [ ] `/reindex`, then `/search WIFI_EVENT_STA_DISCONNECTED`.
- [ ] **Add 2–3 questions using this identifier** to `eval/questions.json` from Lesson 8, with `esp32-wifi.md` as the expected source.

   Example entries to add to the array in `eval/questions.json`. Add a comma after the last existing entry first:
   ```json
   { "question": "What event fires when the ESP32 loses its WiFi connection?", "expectedSources": ["esp32-wifi.md"] },
   { "question": "What does WIFI_EVENT_STA_DISCONNECTED mean?", "expectedSources": ["esp32-wifi.md"] },
   { "question": "Which function reconnects after a WiFi drop?", "expectedSources": ["esp32-wifi.md"] }
   ```
   The first question paraphrases the note, so vector search should find it. The second and third use the identifier and the function name, which is where BM25 should help.

**Observe**
- Does vector search rank the chunk containing the exact identifier where you'd expect, or does a topically-similar-but-wrong chunk (if you have one) outrank it? Look at the actual scores from `/search`, not just the top result.

## Exercise 2 — Implement BM25

**Goal:** a second, independent ranking signal, built from word frequency rather than meaning.

**Do**

- [ ] **Add a BM25 scorer.** Create `Rag/Bm25.cs` — this needs the term frequency of each query word in each chunk, and how rare that word is across all chunks (inverse document frequency). A minimal version:
   ```csharp
   namespace LocalAgent.Rag;

   public static class Bm25
   {
       public static IReadOnlyList<(int Index, double Score)> Score(
           IReadOnlyList<string> documents, string query, double k1 = 1.5, double b = 0.75)
       {
           var tokenizedDocs = documents.Select(Tokenize).ToList();

           // Average() throws on an empty list, so an empty notes folder would crash the command.
           // Return an empty result instead, before any math runs.
           if (tokenizedDocs.Count == 0)
           {
               return Array.Empty<(int Index, double Score)>();
           }

           var avgLen = tokenizedDocs.Average(d => d.Count);
           var queryTerms = Tokenize(query);
           // Document frequency: how many documents contain each query term at all -- rarer
           // terms (lower docFreq) get a bigger idf weight below.
           var df = queryTerms.Distinct().ToDictionary(
               term => term,
               term => tokenizedDocs.Count(d => d.Contains(term)));

           var scores = new List<(int Index, double Score)>();
           for (int i = 0; i < tokenizedDocs.Count; i++)
           {
               var doc = tokenizedDocs[i];
               double score = 0;
               foreach (var term in queryTerms.Distinct())
               {
                   int tf = doc.Count(t => t == term); // term frequency: how many times this word appears in this doc
                   if (tf == 0 || !df.TryGetValue(term, out var docFreq) || docFreq == 0) continue;
                   double idf = Math.Log(1 + (documents.Count - docFreq + 0.5) / (docFreq + 0.5));
                   // Standard BM25 term score: idf weight times a saturating function of term
                   // frequency, normalized against this document's length relative to the average.
                   score += idf * (tf * (k1 + 1)) / (tf + k1 * (1 - b + b * doc.Count / avgLen));
               }
               scores.Add((i, score));
           }
           return scores.OrderByDescending(s => s.Score).ToList();
       }

       // Splits text into lowercase words. Each separator below matters for matching:
       // - '`' (backtick): markdown code marks, so `WIFI_EVENT_STA_DISCONNECTED` becomes the plain word.
       // - '\r': Windows line endings, so the last word of a line does not keep a hidden carriage return.
       // - Forgetting these means an identifier in backticks never matches a plain query, and BM25 returns nothing.
       private static List<string> Tokenize(string text)
           => text.ToLowerInvariant().Split([' ', '\n', '\r', '\t', '.', ',', '(', ')', ':', '"', '`'],
               StringSplitOptions.RemoveEmptyEntries).ToList();
   }
   ```
   **Wiring it into the command helper.** The switch case is only a dispatcher: it calls a method and then `break`s. The label is `"/search-bm25"`, exactly like the other commands. The parser has already lowercased the command, so it matches. Put the case between `/eval` and `default:`:
   ```csharp
   case "/search-bm25":
       await SearchBm25Async(parsedInput.input, context);
       break;
   ```
   The method goes next to `EvalAsync` in `Helpers/CommandHelper.cs`. It needs `using LocalAgent.Rag;` at the top of that file to see `Bm25`:
   ```csharp
   private static Task SearchBm25Async(string query, CommandContext ctx)
   {
       var documents = ctx.Store.All.Select(c => c.Text).ToList();
       var ranked = Bm25.Score(documents, query);

       // Score is 0 for chunks that share no words with the query; skip those.
       foreach (var (index, score) in ranked.Where(r => r.Score > 0).Take(6))
       {
           var chunk = ctx.Store.All[index];
           Console.WriteLine($"[{chunk.SourceFile}] (bm25 {score:F2})\n{chunk.Text}\n---");
       }
       return Task.CompletedTask;
   }
   ```
   Two things to note. `Score` returns the indices of `documents`, which are the same order as `ctx.Store.All`, so `ctx.Store.All[index]` gives back the matching chunk and its `SourceFile`. And the method returns `Task` only so it can be awaited in the switch like `/search`. It doesn't do any async work.
- [ ] **Add a `/search-bm25 <query>` command** to `Program.cs`, scoring over `store.All.Select(c => c.Text).ToList()` and printing the top results the same way `/search` does, with source file and score. Placement: in your command helper (Lesson 3's checkpoint) — one new `case "/search-bm25":` in `CommandHelper.HandleAsync` calling a new private `SearchBm25Async(argument, ctx)` method, next to the `/search`, `/chunks`, `/reindex` and `/eval` cases. The `while` loop doesn't change. (The parser splits on the first space, so `/search-bm25 wifi drop` arrives as command `/search-bm25` with argument `wifi drop`.)
- [ ] `dotnet run`, `/reindex`, then run the identifier query from Exercise 1 through `/search-bm25` and compare its ranking to plain `/search`.

## Exercise 3 — Merge with RRF

**Goal:** combine both signals into one ranking.

**Do**

- [ ] **Implement RRF** in `Rag/VectorStore.cs` or a new `Rag/HybridSearch.cs`: get both rankings (vector `TopK` over the full set, and `Bm25.Score`), then for each chunk sum `1 / (k + rank)` across whichever list(s) it appears in (`k = 60`, ranks 1-based):
   ```csharp
   public static List<(int Index, double RrfScore)> Fuse(
       List<int> vectorRanking, List<int> bm25Ranking, int k = 60)
   {
       var scores = new Dictionary<int, double>();
       void AddRanks(List<int> ranking)
       {
           for (int rank = 0; rank < ranking.Count; rank++)
           {
               var idx = ranking[rank];
               // 1/(k+rank): a chunk's contribution shrinks the further down a list it ranks.
               // Summing across both lists means ranking high in either (or both) helps.
               scores[idx] = scores.GetValueOrDefault(idx) + 1.0 / (k + rank + 1);
           }
       }
       AddRanks(vectorRanking);
       AddRanks(bm25Ranking);
       return scores.OrderByDescending(kv => kv.Value).Select(kv => (kv.Key, kv.Value)).ToList();
   }
   ```
- [ ] **Add a `/search-hybrid <query>` command** that runs both rankings over `store.All`, fuses them, and prints the top results — keep `/search` and `/search-bm25` working too, so you can compare all three side by side. Same placement as the other slash commands: a new `case "/search-hybrid":` in `CommandHelper.HandleAsync` plus a private handler method.
- [ ] **Point your `SearchNotes` tool and `/eval` harness at the hybrid ranking** instead of vector-only (or add a flag to switch between them, so you can run `/eval` both ways without changing code each time). Placement: in `Tools/NoteTools.cs`, this replaces the `var hits = store.TopK(queryVector, k: 6);` line inside `SearchNotes` with a call that also runs BM25 and fuses the two rankings; in `Helpers/CommandHelper.cs`, the equivalent replacement goes inside your `EvalAsync` method, at the same `ctx.Store.TopK(...)` call.
- [ ] Run `/eval` once with vector-only, once with hybrid.

**Observe**
- What's the recall@k delta overall, and specifically on the identifier-based questions you added in Exercise 1?
- Did hybrid search hurt any of your paraphrase-style questions from earlier lessons, or was it a clean win?

**Discuss**
- Why does RRF use rank position rather than directly combining the raw BM25 and cosine-similarity scores?
- What value of `k` in the RRF formula would favor BM25-style exact matches more? What would favor broad semantic recall more?
- Where would a third signal (e.g. a reranker model) fit into this pipeline, and what would it cost?
