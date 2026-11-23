# Lesson 2 — Exercises

Both `llama-server` windows from Lesson 0 must still be running. All file paths are relative to your `LocalAgent` project root.

## Exercise 1 — Minimal vector store

**Goal:** a working, inspectable search index with zero LLM involvement.

**Do**

- [ ] **Create a notes folder.** Inside your project root, create `samples/notes/` and write 5–10 short `.md` files about a topic you know well (so you can judge relevance yourself) — e.g. one file per subtopic, a few paragraphs each. You'll keep adding to this folder throughout the course.
- [ ] **Add an embedding client.** In `Program.cs`, build a second client alongside your chat client, pointed at port 8081:
   ```csharp
   using Microsoft.Extensions.AI;
   using OpenAI;
   using System.ClientModel;

   var embeddingGenerator = new OpenAIClient(
           new ApiKeyCredential("not-needed"),
           new OpenAIClientOptions { Endpoint = new Uri("http://localhost:8081/v1") })
       .GetEmbeddingClient("qwen3-embedding-0.6b")
       .AsIEmbeddingGenerator();
   ```
- [ ] **Create `Rag/Chunk.cs`** to hold one embedded piece of text:
   ```csharp
   namespace LocalAgent.Rag;

   public record Chunk(string SourceFile, string Text, float[] Vector);
   ```
- [ ] **Create `Rag/VectorStore.cs`** with an in-memory list and a `TopK` method using cosine similarity (`System.Numerics.Tensors.TensorPrimitives.CosineSimilarity` does this for you if you add `<PackageReference Include="System.Numerics.Tensors" />` to your `.csproj`):
   ```csharp
   using System.Numerics.Tensors;

   namespace LocalAgent.Rag;

   public class VectorStore
   {
       private readonly List<Chunk> _chunks = [];

       public void Add(Chunk chunk) => _chunks.Add(chunk);

       // Returns the k chunks whose vectors are most similar to queryVector, highest score first.
       public IReadOnlyList<(Chunk Chunk, float Score)> TopK(float[] queryVector, int k)
           => _chunks
               .Select(c => (Chunk: c, Score: TensorPrimitives.CosineSimilarity(c.Vector, queryVector)))
               .OrderByDescending(r => r.Score)
               .Take(k)
               .ToList();

       public IReadOnlyList<Chunk> All => _chunks;
   }
   ```
- [ ] **Index the notes folder at startup**, reading every `.md` file under `samples/notes/` and embedding its full contents as one chunk (don't split yet — that's Lesson 3). Placement: right after your `embeddingGenerator` is built, and before anything involving `chatClient` — this step only depends on the embedding client, and `store` needs to exist, at this same method-level scope, before the `while (true)` input loop starts, since Exercise 2's `/search` command (added inside that loop) reads from it:
   ```csharp
   var store = new VectorStore();
   foreach (var path in Directory.GetFiles("samples/notes", "*.md", SearchOption.AllDirectories))
   {
       var text = await File.ReadAllTextAsync(path);
       var embedding = await embeddingGenerator.GenerateVectorAsync(text);
       store.Add(new Chunk(Path.GetFileName(path), text, embedding.ToArray()));
   }
   ```
- [ ] **Add a `Hash` field to `Chunk`**, so each chunk can carry the hash of the source file it came from (needed for the change-detection step below):
   ```csharp
   public record Chunk(string SourceFile, string Text, float[] Vector, string Hash);
   ```
   This changes the constructor shape — your indexing loop's `new Chunk(...)` call now needs a fourth argument.
- [ ] **Create `Rag/Cacher.cs`** to load and save the index as `index.json` in your project root. `Retrieve` deserializes straight into `List<Chunk>` (it works without any extra setup because `Chunk` is a record with a single constructor, so `System.Text.Json` can match its properties by name) and wraps the results in a `VectorStore`; `Save` writes `store.All` back out:
   ```csharp
   namespace LocalAgent.Rag;

   public static class Cacher
   {
       private const string FileName = "index.json";

       public static VectorStore Retrieve()
       {
           var store = new VectorStore();
           if (!File.Exists(FileName))
               return store; // nothing cached yet -- first run

           var json = File.ReadAllText(FileName);
           // Deserialize into List<Chunk>, not VectorStore directly -- see the note below the block.
           var chunks = System.Text.Json.JsonSerializer.Deserialize<List<Chunk>>(json) ?? [];
           foreach (var chunk in chunks)
               store.Add(chunk);

           return store;
       }

       public static void Save(VectorStore store)
       {
           var json = System.Text.Json.JsonSerializer.Serialize(store.All);
           File.WriteAllText(FileName, json);
       }
   }
   ```
   (Don't be tempted to `Deserialize<VectorStore>` directly — `VectorStore`'s chunk list is private with no public setter or matching constructor, so `System.Text.Json` would silently hand you back an empty store instead of failing loudly. Deserializing into `List<Chunk>` first and adding each one through `VectorStore.Add` sidesteps that.)
- [ ] **Update your indexing loop to use the cache.** Placement: this *replaces* the simple indexing loop from the bullet above, in the same spot — load the cache first, compute each file's hash, and only call the embedding model for files that are new or changed:
   ```csharp
   var cached = Cacher.Retrieve();
   var cachedByFile = cached.All.ToDictionary(c => c.SourceFile);

   var store = new VectorStore();
   foreach (var path in Directory.GetFiles("samples/notes", "*.md", SearchOption.AllDirectories))
   {
       var fileName = Path.GetFileName(path);
       var bytes = await File.ReadAllBytesAsync(path);
       var hash = Convert.ToHexString(System.Security.Cryptography.SHA256.HashData(bytes));

       if (cachedByFile.TryGetValue(fileName, out var old) && old.Hash == hash)
       {
           store.Add(old); // unchanged -- reuse the cached chunk and its vector, skip embedding
       }
       else
       {
           var text = await File.ReadAllTextAsync(path);
           var embedding = await embeddingGenerator.GenerateVectorAsync(text);
           store.Add(new Chunk(fileName, text, embedding.ToArray(), hash)); // new or changed
       }
   }
   Cacher.Save(store);
   ```
- [ ] **Run `dotnet run` twice in a row** without changing any notes. The second run should be noticeably faster to reach the prompt, since nothing gets re-embedded. Then edit one note's content and run again — only that file's chunk should get re-embedded.

**Observe**
- Delete `index.json` and run once — does it rebuild cleanly from nothing? Edit one note and run again — does only that file get re-embedded, going by how long the startup takes?

## Exercise 2 — `/search` as a standalone command

**Goal:** a tool for looking at retrieval quality directly, with no LLM involved.

**Do**

- [ ] **Add a `/search <text>` command** to your `Program.cs` input loop. Placement: inside the `while (true)` loop, right after you read `input` and check it isn't empty, but *before* the line that calls `loop.AskAsync(input)` — this command needs to intercept the input and `continue` the loop instead of ever reaching the agent:
   ```csharp
   if (input.StartsWith("/search "))
   {
       var query = input["/search ".Length..];
       var queryVector = (await embeddingGenerator.GenerateVectorAsync(query)).ToArray();
       foreach (var (chunk, score) in store.TopK(queryVector, k: 6))
       {
           Console.WriteLine($"{score:F3}  {chunk.SourceFile}  \"{chunk.Text[..Math.Min(80, chunk.Text.Length)]}…\"");
       }

       continue; // AFTER the foreach's closing brace, inside the if: skips to the next prompt instead of sending "/search ..." to the agent
   }
   ```
   (Heads-up: this `if` block is a stepping stone. In Lesson 3, once you have a second and third command, you'll fold all of them into one command helper so you don't keep repeating this pattern.)

   (If `continue;` ends up *inside* the `foreach` braces, it only skips to the next result — the loop then falls through to `loop.AskAsync(input)` and the agent answers your `/search` command too. If you see an `[llm]` line after your search results, this is why.)
- [ ] **Run `dotnet run`**, then:
   - `/search` using the exact wording from one of your notes.
   - `/search` again with a paraphrase — different words, same meaning.
   - `/search` with something totally unrelated to anything in your notes.

**Observe**
- Is there a clear score gap between a good match and the unrelated query? How big, in the numbers you actually see printed?
- Did the paraphrase still find the right note, despite no shared words?

**Discuss**
- What would a reasonable `MinScore` cutoff be, based on the numbers you saw? What's the risk of setting it too high?
- When would plain keyword search (e.g. matching an exact error code like `WIFI_EVENT_STA_DISCONNECTED`) beat vector search outright?
