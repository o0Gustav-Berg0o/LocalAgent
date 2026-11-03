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
           var avgLen = tokenizedDocs.Average(d => d.Count);
           var queryTerms = Tokenize(query);
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
                   int tf = doc.Count(t => t == term);
                   if (tf == 0 || !df.TryGetValue(term, out var docFreq) || docFreq == 0) continue;
                   double idf = Math.Log(1 + (documents.Count - docFreq + 0.5) / (docFreq + 0.5));
                   score += idf * (tf * (k1 + 1)) / (tf + k1 * (1 - b + b * doc.Count / avgLen));
               }
               scores.Add((i, score));
           }
           return scores.OrderByDescending(s => s.Score).ToList();
       }

       private static List<string> Tokenize(string text)
           => text.ToLowerInvariant().Split([' ', '\n', '\t', '.', ',', '(', ')', ':', '"'],
               StringSplitOptions.RemoveEmptyEntries).ToList();
   }
   ```
- [ ] **Add a `/search-bm25 <query>` command** to `Program.cs`, scoring over `store.All.Select(c => c.Text).ToList()` and printing the top results the same way `/search` does, with source file and score.
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
               scores[idx] = scores.GetValueOrDefault(idx) + 1.0 / (k + rank + 1);
           }
       }
       AddRanks(vectorRanking);
       AddRanks(bm25Ranking);
       return scores.OrderByDescending(kv => kv.Value).Select(kv => (kv.Key, kv.Value)).ToList();
   }
   ```
- [ ] **Add a `/search-hybrid <query>` command** that runs both rankings over `store.All`, fuses them, and prints the top results — keep `/search` and `/search-bm25` working too, so you can compare all three side by side.
- [ ] **Point your `SearchNotes` tool and `/eval` harness at the hybrid ranking** instead of vector-only (or add a flag to switch between them, so you can run `/eval` both ways without changing code each time).
- [ ] Run `/eval` once with vector-only, once with hybrid.

**Observe**
- What's the recall@k delta overall, and specifically on the identifier-based questions you added in Exercise 1?
- Did hybrid search hurt any of your paraphrase-style questions from earlier lessons, or was it a clean win?

**Discuss**
- Why does RRF use rank position rather than directly combining the raw BM25 and cosine-similarity scores?
- What value of `k` in the RRF formula would favor BM25-style exact matches more? What would favor broad semantic recall more?
- Where would a third signal (e.g. a reranker model) fit into this pipeline, and what would it cost?
