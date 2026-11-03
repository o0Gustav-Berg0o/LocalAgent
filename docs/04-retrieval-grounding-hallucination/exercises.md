# Lesson 4 — Exercises

Both `llama-server` windows must still be running. All paths are relative to your project root.

## Exercise 1 — Wire up `search_notes`

**Goal:** retrieval becomes something the model chooses to invoke, not something you always inject.

**Do**

- [ ] **Create `Tools/NoteTools.cs`**, turning your `/search` logic from Lesson 2 into a tool method instead of a console command:
   ```csharp
   using System.ComponentModel;
   using LocalAgent.Rag;
   using Microsoft.Extensions.AI;

   namespace LocalAgent.Tools;

   public class NoteTools(VectorStore store, IEmbeddingGenerator<string, Embedding<float>> embeddingGenerator)
   {
       [Description("Search the user's own notes for passages relevant to a question. Always call this FIRST for any question that might be about the user's notes, before answering from your own knowledge.")]
       public async Task<string> SearchNotes(
           [Description("A short search query capturing what to look for, not the user's full original sentence.")]
           string query)
       {
           var queryVector = (await embeddingGenerator.GenerateVectorAsync($"Instruct: Given a question, retrieve relevant passages\nQuery:{query}")).ToArray();
           var hits = store.TopK(queryVector, k: 6);
           if (hits.Count == 0) return "No relevant passages found.";

           return string.Join("\n---\n", hits.Select(h => $"[{h.Chunk.HeadingPath}] (score {h.Score:F2})\n{h.Chunk.Text}"));
       }
   }
   ```
   Note the `Instruct: ... \nQuery:` prefix on the query embedding — that's the asymmetric-search instruction from Lesson 2's preview. Your *documents* stay embedded as plain `heading path + text`, with no prefix; only the *query* side gets it.
- [ ] **Register it as a tool** in `Program.cs`, alongside your existing tools, and add it to the `AgentLoop`'s tool list:
   ```csharp
   var noteTools = new NoteTools(store, embeddingGenerator);
   AIFunction[] tools =
   [
       AIFunctionFactory.Create(UtilityTools.GetDateTime),
       AIFunctionFactory.Create(UtilityTools.Calculate),
       AIFunctionFactory.Create(noteTools.SearchNotes),
   ];
   ```
- [ ] **Add a system prompt.** Pass it as the first message in your `AgentLoop`'s history (or as a constructor parameter you prepend before the user's first message):
   ```csharp
   private static readonly ChatMessage SystemPrompt = new(ChatRole.System, """
       You are a helpful assistant with access to the user's personal notes.
       For ANY question that might relate to the user's notes, call search_notes FIRST, before answering.
       Base your answer only on what search_notes returns. If it returns "No relevant passages found"
       or nothing relevant to the question, say plainly that your notes don't cover this — do not guess
       from your own training data. Cite the source as [file > heading] when you use a passage.
       """);
   ```
- [ ] **Run `dotnet run`** with `/trace` behavior visible (your `[tool]` line from Lesson 1 already shows this) and try the questions in Observe below.

**Observe**
- Ask a question whose answer is in your notes. Check the trace: did it call `search_notes`? With what query text — the same as your question, or something reformulated?
- Ask a pure greeting ("Hi, who are you?"). Did it call `search_notes` anyway? Should it have?

## Exercise 2 — Break asymmetric search on purpose

**Goal:** feel what a "quiet" RAG bug looks like — no error, just worse results.

**Do**

- [ ] Pick three real questions about your notes. Run each through `/search` (your Lesson 2 command, which calls the same embedding path) and write down the top result's score for each.
- [ ] **Temporarily remove the query prefix** in `NoteTools.SearchNotes` — embed just `query` instead of `$"Instruct: ...\nQuery:{query}"`. No `/reindex` needed; only how the *query* gets embedded changed, not your stored document vectors.
- [ ] Re-run the exact same three `/search` queries.
- [ ] Restore the prefix and confirm scores return to their original levels.

**Observe**
- Did rankings change? Did scores change even when the top result stayed the same?

## Exercise 3 — Make "I don't know" a correct answer

**Goal:** separate grounded answers from hallucinated ones.

**Do**

- [ ] Ask about a topic you genuinely never wrote a note about. Does the model say it doesn't know, or does it confidently answer anyway from training data?
- [ ] **Add a `MinScore` cutoff** inside `SearchNotes`, before building the result string:
   ```csharp
   var relevant = hits.Where(h => h.Score >= 0.3f).ToList(); // tune this threshold against what you saw in Lesson 2
   if (relevant.Count == 0) return "No relevant passages found.";
   ```
- [ ] Repeat the missing-topic question — `search_notes` should now explicitly report nothing relevant, rather than returning weak matches.
- [ ] **Now soften the system prompt** on purpose: remove the "do not guess from your own training data" sentence, and ask the missing-topic question again.

**Observe**
- With the strict prompt and the `MinScore` cutoff, does the model actually say it doesn't know? With the softened prompt, does the same question get a confident, unsupported answer instead?

**Discuss**
- What's actually worse for a user: a confident wrong answer, or an overly cautious "I don't know" on something the notes did contain?
- A similarity score is not a truth score. What exactly does a high cosine similarity tell you, and what does it *not* tell you?
