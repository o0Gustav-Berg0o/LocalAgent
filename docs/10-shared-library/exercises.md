# Lesson 10 — Exercises

Both `llama-server` windows must still be running.

**Prerequisites:** Lesson 9 is complete, so `/search-hybrid` runs. Work in a clean git state, so every step can be undone with a single revert.

**Rule for this lesson:** after every move, build, run `/eval`, and compare with the baseline from Exercise 1. If a number changes, undo that step before doing anything else.

**Namespaces stay the same during this lesson.** The library's assembly is called `LocalAgent.Core`, but the files keep their namespaces (`LocalAgent.Rag`, `LocalAgent.Helpers`). C# does not require a namespace to match the assembly name, so the `using` lines in the console app keep working without edits. Renaming namespaces is a separate, optional step at the end of the course. Do not combine it with this refactor.

## Exercise 1 — Record the baseline

**Goal:** numbers you can compare against after every step.

**Do**

- [ ] Run `/reindex`, then `/eval`. Write down the total and the full PASS/FAIL list.
- [ ] Run `/eval` a second time without changing anything. Compare the two runs.
- [ ] Run `/search-hybrid WIFI_EVENT_STA_DISCONNECTED` and note the top result and its score.

**Record it in a table you can compare against later:**

```text
Run              recall@6   Failing questions          Top result for WIFI_EVENT_STA_DISCONNECTED
baseline 1       __ %       ...                        wifi.md, score __
baseline 2       __ %       ...                        wifi.md, score __
```

**Observe**
- If the two `/eval` runs differ with nothing edited, something non-deterministic is in the pipeline. Find it before refactoring, or every later comparison is meaningless.

## Exercise 2 — Create the library project

**Goal:** an empty class library that the console app can reference, and that still builds.

**Background:** a *solution* (`LocalAgent.slnx`) can contain several *projects*. Each project has its own `.csproj` file. The console app is one project. Here you add a second one, a *class library*: it compiles into a `.dll` that other projects can reference. It has no `Main` method and cannot run on its own.

**Do**

- [ ] **Create the project.** From the folder that contains `LocalAgent.csproj`, run:

   ```bash
   dotnet new classlib -n LocalAgent.Core -f net10.0
   ```

   This creates a folder `LocalAgent.Core` with `LocalAgent.Core.csproj` and a placeholder file `Class1.cs`. Delete `Class1.cs`. It is only a template example.

- [ ] **Add it to the solution**, so Visual Studio and `dotnet build` see both projects:

   ```bash
   dotnet sln LocalAgent.slnx add LocalAgent.Core/LocalAgent.Core.csproj
   ```

   If your SDK rejects `.slnx` with this command, add the project in Visual Studio instead: right-click the solution, then **Add > Existing Project**, and choose `LocalAgent.Core.csproj`.

- [ ] **Make the console app reference it.** This is the arrow that matters: the app depends on the library, never the reverse.

   ```bash
   dotnet add LocalAgent.csproj reference LocalAgent.Core/LocalAgent.Core.csproj
   ```

   The command writes this into `LocalAgent.csproj`. You can check it by opening the file:

   ```xml
   <ItemGroup>
     <ProjectReference Include="..\LocalAgent.Core\LocalAgent.Core.csproj" />
   </ItemGroup>
   ```

- [ ] **Copy the packages the moved code needs into the library.** The library needs its own package references, because the app's references do not flow into it. Use the same versions as `LocalAgent.csproj`. Check the versions there before you run these:

   ```bash
   dotnet add LocalAgent.Core package Markdig --version 1.6.0
   dotnet add LocalAgent.Core package Microsoft.Extensions.AI.Abstractions --version 10.11.0
   ```

   `Markdig` is used by the chunker. `Microsoft.Extensions.AI.Abstractions` provides `IEmbeddingGenerator`, used by the indexer and the search code. Leave the console app's packages where they are.

- [ ] **Build.** Run `dotnet build` from the solution folder. It should compile, because nothing has moved yet. The library is empty, and the app is unchanged.

- [ ] **Run `/eval`** and compare with Exercise 1. The numbers must match.

**Observe**
- Which project depends on which? Open both `.csproj` files and write down the direction of the `ProjectReference`.
- Why must the library not reference `LocalAgent.csproj`? What would happen to the library's reuse if it did?

## Exercise 3 — Move the data types and the storage

**Goal:** the chunk record, the chunker, the vector store and the cache live in the library. The app still builds without changing its `using` lines.

**Background:** these four files depend only on each other and on `Markdig` and the AI abstractions. Move them together, so no file is left pointing at a type that stayed behind:

| File | What it does |
|---|---|
| `Rag/Chunk.cs` | the record that holds one piece of text, its vector, hash and paths |
| `Rag/MarkdownChunker.cs` | splits a markdown file into heading-based chunks |
| `Rag/VectorStore.cs` | holds the chunks and finds the most similar ones |
| `Rag/Cacher.cs` | loads and saves `index.json` |

**Do**

- [ ] **Move the files with `git mv`**, so git keeps their history. Run from the solution folder:

   ```bash
   mkdir LocalAgent.Core/Rag
   git mv Rag/Chunk.cs LocalAgent.Core/Rag/
   git mv Rag/MarkdownChunker.cs LocalAgent.Core/Rag/
   git mv Rag/VectorStore.cs LocalAgent.Core/Rag/
   git mv Rag/Cacher.cs LocalAgent.Core/Rag/
   ```

   Do not edit the namespace line at the top of each file. It stays `namespace LocalAgent.Rag;`.

- [ ] **Check the access level of each type the app uses.** The app can only see `public` types from another project. Open each moved file and check the class and record declarations. For example, this is the kind of change you may need:

   ```csharp
   // Before: only code inside the same project can use this.
   internal class Cacher { /* ... */ }

   // After: any project that references LocalAgent.Core can use it.
   public class Cacher { /* ... */ }
   ```

   Change only what the app or the tests call. Anything used only inside its own file stays as it is.

- [ ] **Build.** Run `dotnet build`. Read the errors in this order:

   | Error | Meaning | Fix |
   |---|---|---|
   | `CS0246` type or namespace not found | a package or `using` is missing | add the `using`, or add the package to `LocalAgent.Core` |
   | `CS0122` ... is inaccessible | the type is `internal` but the app uses it | make it `public` |
   | `MSB3021` / `MSB3027` cannot copy the DLL | a running process holds the file | stop the `LocalAgentMcp` process, then build again |

   You may get no errors at all. That is a good sign: it means the files only needed what the library already has.

- [ ] **Run `/reindex` and `/eval`.** Compare with Exercise 1.

**Observe**
- Which types did you have to make `public`? Write the list. Each one is now a promise to other projects.

## Exercise 4 — Move the ranking code

**Goal:** the keyword ranking, the fusion of two rankings, and the query prefix live in the library. This is the one place where the prefix is defined.

**Do**

- [ ] **Move these three files**, with `git mv`, from the same folder layout as before:

   ```bash
   git mv Rag/Bm25.cs LocalAgent.Core/Rag/
   git mv Rag/HybridSearch.cs LocalAgent.Core/Rag/
   git mv Rag/QueryTools.cs LocalAgent.Core/Rag/
   ```

- [ ] **Check that `QueryTools` is `public`.** `/search` and `/eval` in the app call `QueryTools.Prefix`, so it must be.

- [ ] **Build, then run `/eval` and `/search-hybrid`.** Compare with Exercise 1. `/search-hybrid` is the test for `Bm25` and `HybridSearch`, which you just moved.

**Observe**
- `/search`, `/eval` and the agent's search all call `QueryTools.Prefix`. Where else in the app is the instruction text typed out? If you find one, that copy will drift. Note it for the course's "fix the duplicate" step.

## Exercise 5 — Move the search logic, and keep the adapter in the app

**Goal:** the search itself is a library method that takes a query. `NoteTool` in the app is only the part the model sees: it describes the tool, calls the library, and formats the answer.

**Background:** the difference is important. The library method returns data (chunks and scores). The adapter returns text, for the model. A library method must not know about tools, prompts or the console, so that a second app (Lesson 11) can use it unchanged.

**Do**

- [ ] **Create `LocalAgent.Core/Rag/Search.cs`.** Use the same namespace as its neighbours, `LocalAgent.Rag`:

   ```csharp
   using Microsoft.Extensions.AI;

   namespace LocalAgent.Rag;

   public static class Search
   {
       // Takes a question and returns the best-matching chunks with their scores.
       // No tools, no prompts, no console output: data in, data out.
       public static async Task<List<(Chunk Chunk, float Score)>> FindAsync(
           VectorStore store,
           IEmbeddingGenerator<string, Embedding<float>> embeddings,
           string query,
           int k = 6)
       {
           // The prefix is added here, once, so every caller gets the same query text.
           var vector = (await embeddings.GenerateVectorAsync(QueryTools.Prefix(query))).ToArray();
           return store.TopK(vector, k);
       }
   }
   ```

   Read the code line by line. `FindAsync` is `async` because embedding is a network call. It returns a list of tuples, `(Chunk, Score)`, which the caller can read by name: `hit.Chunk.Text`, `hit.Score`.

- [ ] **Make `NoteTool` call the library method.** In `Helpers/NoteTool.cs`, replace the three lines that compute the vector and call `TopK` with one call to `Search.FindAsync`, then keep the threshold and the formatting in `NoteTool`. The shape of the method after the change:

   ```csharp
   public async Task<string> SearchNotes(string query)
   {
       var store = getStore();
       // The library does the search. NoteTool decides what the model sees.
       var hits = await Search.FindAsync(store, embeddingGenerator, query, k: 6);
       var relevant = hits.Where(h => h.Score >= 0.3f).ToList();
       if (relevant.Count == 0) return "No relevant passage found";
       return string.Join("\n---\n", relevant.Select(h => $"[{h.Chunk.HeadingPath}] (score {h.Score:F2})\n{h.Chunk.Text}"));
   }
   ```

   `NoteTool` still owns the threshold, the wording of the "not found" message, and the output format. Those are the adapter's decisions, so they stay in the app. The `using LocalAgent.Rag;` line at the top of `NoteTool.cs` is what makes `Search` visible.

- [ ] **Build, then run `/eval` and ask the agent one notes question.** Compare with Exercise 1. The `[tool]` trace line should look the same as before the change.

**Observe**
- Could a second app use `Search.FindAsync` without any change to it? Try to name what that app would need to pass in.
- If you were tempted to add the threshold to `Search`, what would be wrong with that? Think about what a different app might want.

## Exercise 6 — Move the indexer and add the factory method

**Goal:** one method builds a ready store for a notes folder. Both apps call it, so the indexing setup exists in one place.

**Do**

- [ ] **Move the indexer into the library**, keeping its namespace `LocalAgent.Helpers`:

   ```bash
   mkdir LocalAgent.Core/Helpers
   git mv Helpers/Indexer.cs LocalAgent.Core/Helpers/
   ```

- [ ] **Make `Indexer` public.** It is `internal` today, so the app could not see it from the library. Change its declaration from `internal class Indexer` to `public class Indexer`. Its methods must be `public` too, because `SearchFactory` calls `CreateAsync`.

- [ ] **Create `LocalAgent.Core/Helpers/SearchFactory.cs`**, in the same namespace as `Indexer`:

   ```csharp
   using LocalAgent.Rag;
   using Microsoft.Extensions.AI;

   namespace LocalAgent.Helpers;

   public static class SearchFactory
   {
       // Builds the store for one notes folder. The console app and the MCP server both call
       // this method, so the indexing setup exists in exactly one place.
       public static async Task<VectorStore> BuildStoreAsync(
           string notesFolder,
           IEmbeddingGenerator<string, Embedding<float>> embeddings)
       {
           var indexer = new Indexer();
           return await indexer.CreateAsync(notesFolder, embeddings);
       }
   }
   ```

   *Why a static method:* the factory holds no state. Each app keeps its own store, and this method only builds one.

- [ ] **Call it from `Program.cs`.** Replace the lines that create the `Indexer` and call `CreateAsync` with one call:

   ```csharp
   var store = await SearchFactory.BuildStoreAsync(folderPath, embeddingGenerator);
   ```

   `folderPath` and `embeddingGenerator` are the variables you already have in `Program.cs`. The `using LocalAgent.Helpers;` line stays.

- [ ] **Build, then run `/reindex` and `/eval`.** The numbers must match Exercise 1.

**Observe**
- `Program.cs` should be shorter. What did it lose, and what did it keep?
- `CommandHelper.Reindex` also creates an `Indexer`. Should it call the factory too? What would change if it did?

## Exercise 7 — Prove nothing changed

**Goal:** evidence, not a feeling, that the refactor is safe.

**Do**

- [ ] Run `/eval` and compare the full PASS/FAIL list with Exercise 1, question by question.
- [ ] Run `/search-hybrid WIFI_EVENT_STA_DISCONNECTED` and compare the top result and score with Exercise 1.
- [ ] Write a short note (five lines or fewer) listing what moved, what stayed in the app, and the two comparisons.

## Discuss

- Why is `/eval` a better check for this refactor than running the app once and seeing that it works?
- The library now has a public surface. Which parts would you keep stable, and which can change freely?
- Lesson 11 will host the same search in a separate process. What do you already have in place for that to work, and what is still missing?
- Why did we keep the namespaces unchanged? What would a rename have cost, and when would it be worth doing?
