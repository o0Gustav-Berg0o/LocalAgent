# Lesson 2 — Exercises

Both `llama-server` windows from Lesson 0 must still be running. All file paths are relative to your `LocalAgent` project root.

## Exercise 1 — Minimal vector store

**Goal:** a working, inspectable search index with zero LLM involvement.

**Do**

1. **Create a notes folder.** Inside your project root, create `samples/notes/` and write 5–10 short `.md` files about a topic you know well (so you can judge relevance yourself) — e.g. one file per subtopic, a few paragraphs each. You'll keep adding to this folder throughout the course.
2. **Add an embedding client.** In `Program.cs`, build a second client alongside your chat client, pointed at port 8081:
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
3. **Create `Rag/Chunk.cs`** to hold one embedded piece of text:
   ```csharp
   namespace LocalAgent.Rag;

   public record Chunk(string SourceFile, string Text, float[] Vector);
   ```
4. **Create `Rag/VectorStore.cs`** with an in-memory list and a `TopK` method using cosine similarity (`System.Numerics.Tensors.TensorPrimitives.CosineSimilarity` does this for you if you add `<PackageReference Include="System.Numerics.Tensors" />` to your `.csproj`):
   ```csharp
   using System.Numerics.Tensors;

   namespace LocalAgent.Rag;

   public class VectorStore
   {
       private readonly List<Chunk> _chunks = [];

       public void Add(Chunk chunk) => _chunks.Add(chunk);

       public IReadOnlyList<(Chunk Chunk, float Score)> TopK(float[] queryVector, int k)
           => _chunks
               .Select(c => (Chunk: c, Score: TensorPrimitives.CosineSimilarity(c.Vector, queryVector)))
               .OrderByDescending(r => r.Score)
               .Take(k)
               .ToList();

       public IReadOnlyList<Chunk> All => _chunks;
   }
   ```
5. **Index the notes folder at startup**, reading every `.md` file under `samples/notes/` and embedding its full contents as one chunk (don't split yet — that's Lesson 3):
   ```csharp
   var store = new VectorStore();
   foreach (var path in Directory.GetFiles("samples/notes", "*.md", SearchOption.AllDirectories))
   {
       var text = await File.ReadAllTextAsync(path);
       var embedding = await embeddingGenerator.GenerateVectorAsync(text);
       store.Add(new Chunk(Path.GetFileName(path), text, embedding.ToArray()));
   }
   ```
6. **Add JSON persistence.** Serialize `store.All` to `index.json` after indexing, and on startup, check each note file's content hash against what's stored; only re-embed files that are new or changed. (`System.Security.Cryptography.SHA256` over the file bytes gives you a hash; store it alongside each chunk in the JSON, keyed by source file.) You don't need this working perfectly yet — getting the skip-unchanged-files logic right is itself worth sitting with for a bit.

## Exercise 2 — `/search` as a standalone command

**Goal:** a tool for looking at retrieval quality directly, with no LLM involved.

**Do**

1. **Add a `/search <text>` command** to your `Program.cs` input loop (check for it before passing input to the agent loop):
   ```csharp
   if (input.StartsWith("/search "))
   {
       var query = input["/search ".Length..];
       var queryVector = (await embeddingGenerator.GenerateVectorAsync(query)).ToArray();
       foreach (var (chunk, score) in store.TopK(queryVector, k: 6))
           Console.WriteLine($"{score:F3}  {chunk.SourceFile}  \"{chunk.Text[..Math.Min(80, chunk.Text.Length)]}…\"");
       continue;
   }
   ```
2. **Run `dotnet run`**, then:
   - `/search` using the exact wording from one of your notes.
   - `/search` again with a paraphrase — different words, same meaning.
   - `/search` with something totally unrelated to anything in your notes.

**Observe**
- Is there a clear score gap between a good match and the unrelated query? How big, in the numbers you actually see printed?
- Did the paraphrase still find the right note, despite no shared words?

**Discuss**
- What would a reasonable `MinScore` cutoff be, based on the numbers you saw? What's the risk of setting it too high?
- When would plain keyword search (e.g. matching an exact error code like `WIFI_EVENT_STA_DISCONNECTED`) beat vector search outright?
