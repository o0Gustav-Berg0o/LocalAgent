# Lesson 3 — Exercises

Both `llama-server` windows must still be running. All paths are relative to your project root.

## Exercise 1 — Structure-aware chunker

**Goal:** split documents without ever cutting a block in half.

**Do**

- [ ] **Add Markdig**, the markdown parser you'll use to find block boundaries:
   ```powershell
   dotnet add package Markdig
   ```
- [ ] **Create `Rag/MarkdownChunker.cs`.** The shape of the algorithm: parse the file into a `MarkdownDocument` with `Markdig.Markdown.Parse(text)`; walk its top-level blocks in order; whenever you hit a `HeadingBlock`, flush whatever's in your buffer as a chunk (with the heading path *before* this heading), then update the heading path (push this heading's text onto a stack, popping anything at the same or deeper level first); for every other block type, append its original text to a buffer, and if adding it would push the buffer past a `MaxChunkChars` limit (start with `2000`), flush the buffer as a chunk first, then start a new one with this block. Skip `YamlFrontMatterBlock` entirely. At the end, flush whatever's left in the buffer (skip it if it's empty — that's the "heading with no content" case).
   ```csharp
   namespace LocalAgent.Rag;

   public record Chunk2(string HeadingPath, string Text); // rename/merge with your Lesson 2 Chunk, which by now also carries a Vector and a Hash

   public static class MarkdownChunker
   {
       public static List<Chunk2> Chunk(string fileName, string text, int maxChunkChars = 2000)
       {
           var document = Markdig.Markdown.Parse(text); // parse once into a tree of top-level blocks

           var chunks = new List<Chunk2>();

           // The heading hierarchy we're currently nested under, e.g. [(1, "Setup"), (2, "Prerequisites")].
           // A level-2 heading replaces any previous level-2 (or deeper) entry, but keeps level-1 entries above it.
           var headingStack = new List<(int Level, string Text)>();

           var buffer = new System.Text.StringBuilder(); // text accumulated for the chunk currently being built

           // Turns whatever's in `buffer` into a finished chunk, then clears it so the next block
           // starts a fresh one. A no-op if nothing's been accumulated (e.g. two headings in a row).
           void Flush()
           {
               if (buffer.Length == 0) return;
               var path = string.Join(" > ", new[] { fileName }.Concat(headingStack.Select(h => h.Text)));
               chunks.Add(new Chunk2(path, buffer.ToString().Trim()));
               buffer.Clear();
           }

           foreach (var block in document)
           {
               if (block is Markdig.Extensions.Yaml.YamlFrontMatterBlock) continue; // metadata, not content

               if (block is Markdig.Syntax.HeadingBlock heading)
               {
                   Flush(); // this heading ends whatever chunk was building under the previous one

                   // Drop any stack entries at this heading's level or deeper — a new "## Foo" replaces
                   // a previous "## Bar" and also clears out any "### Baz" nested under that old "## Bar".
                   headingStack.RemoveAll(h => h.Level >= heading.Level);

                   // Pull the heading's raw source text via its span, then strip the leading "#"s and space.
                   var headingText = text[heading.Span.Start..(heading.Span.End + 1)].TrimStart('#', ' ');
                   headingStack.Add((heading.Level, headingText));
                   continue; // a heading itself never becomes chunk content
               }

               // Any non-heading block (paragraph, list, code fence, table, ...) — grab its original text
               // whole, by span, so we never reconstruct or reformat it.
               var blockText = text[block.Span.Start..(block.Span.End + 1)];

               // If this block would push the buffer past the limit, flush what's already there FIRST,
               // so the block that overflowed starts the next chunk instead of being dropped or split.
               if (buffer.Length + blockText.Length > maxChunkChars) Flush();

               buffer.AppendLine(blockText); // always append — this line runs whether or not we just flushed
           }

           Flush(); // the loop ends with content still in the buffer unless the last block was a heading
           return chunks;
       }
   }
   ```
   (Treat this as a starting skeleton, not a finished implementation — check the Markdig API docs for the exact block/span member names in the version you installed, and adjust.)
- [ ] **Add a `/chunks <file>` command** to `Program.cs`. Placement: inside your `while (true)` input loop, alongside your existing `/search` check from Lesson 2 (either `if`/`else if` branch — order between the two doesn't matter, since each checks for its own distinct prefix), still before the line that calls `loop.AskAsync(input)`:
   ```csharp
   if (input.StartsWith("/chunks "))
   {
       var fileName = input["/chunks ".Length..].Trim();
       var path = Path.Combine("samples/notes", fileName);
       var text = await File.ReadAllTextAsync(path);
       foreach (var chunk in LocalAgent.Rag.MarkdownChunker.Chunk(fileName, text))
       {
           Console.WriteLine($"[{chunk.HeadingPath}]\n{chunk.Text}\n---");
       }

       continue; // AFTER the foreach's closing brace, inside the if -- same rule as /search in Lesson 2
   }
   ```
- [ ] **Run `dotnet run`** and try `/chunks <one of your note filenames>.md`.

**Observe**
- Does any chunk cut a code block, list, or table in half? (It shouldn't — if it does, check your block-span handling.)

## Checkpoint — Tidy the input loop with a command helper

You now have two slash commands (`/search`, `/chunks`) as separate `if` blocks, `/reindex` is next, and Lessons 8 and 9 add three more. Each new block repeats the same prefix-slicing and the same `continue;` — the exact line that's easy to put in the wrong place. Do this small refactor **now, before `/reindex`**, so every later command is "add one `case` and one method" instead of "add another `if` block". No new behavior; `/search` and `/chunks` must work exactly as before.

**Goal:** one place that parses a slash command and dispatches it, so `Program.cs`'s loop has a single `if` for all commands.

**Do**

- [ ] **Create `Helpers/CommandContext.cs`** — the shared state the command handlers need, so they don't have to live inside `Main`. `Store` has a setter because `/reindex` (next exercise) replaces it:
   ```csharp
   using LocalAgent.Rag;
   using Microsoft.Extensions.AI;

   namespace LocalAgent.Helpers;

   public class CommandContext
   {
       public required VectorStore Store { get; set; }
       public required IEmbeddingGenerator<string, Embedding<float>> Embeddings { get; init; }
       public required string NotesFolder { get; init; }
   }
   ```
- [ ] **Create `Helpers/CommandHelper.cs`** with a parser and a dispatcher. The parser only *splits* text — it does no I/O and knows no command names, so you can test it by eye. The dispatcher is the one place that knows which commands exist:
   ```csharp
   using LocalAgent.Rag;
   using Microsoft.Extensions.AI; // GenerateVectorAsync is an extension method from this namespace

   namespace LocalAgent.Helpers;

   public static class CommandHelper
   {
       // Splits "/search car tuning" into ("/search", "car tuning"). Pure string work.
       public static (bool IsCommand, string Command, string Argument) Parse(string input)
       {
           if (!input.StartsWith('/'))
               return (false, "", ""); // plain chat -- not our business

           int space = input.IndexOf(' ');
           if (space == -1)
               return (true, input.Trim().ToLowerInvariant(), ""); // "/reindex": no arguments

           // Only the FIRST space separates command from argument, so "/search car tuning"
           // keeps all of "car tuning" as the argument.
           return (true, input[..space].ToLowerInvariant(), input[(space + 1)..].Trim());
       }

       // Returns true if the line was a slash command (handled here), false if it should go to the agent.
       public static async Task<bool> HandleAsync(string input, CommandContext ctx)
       {
           var (isCommand, command, argument) = Parse(input);
           if (!isCommand) return false;

           switch (command)
           {
               case "/search": await SearchAsync(argument, ctx); break;
               case "/chunks": await ChunksAsync(argument, ctx); break;
               default: Console.WriteLine($"Unknown command: {command}"); break;
           }
           return true; // handled -- even an unknown command must not reach the agent
       }

       private static async Task SearchAsync(string query, CommandContext ctx)
       {
           if (string.IsNullOrWhiteSpace(query))
           {
               Console.WriteLine("Usage: /search <text>");
               return;
           }

           var queryVector = (await ctx.Embeddings.GenerateVectorAsync(query)).ToArray();
           foreach (var (chunk, score) in ctx.Store.TopK(queryVector, k: 6))
           {
               Console.WriteLine($"{score:F3}  {chunk.SourceFile}  \"{chunk.Text[..Math.Min(80, chunk.Text.Length)]}…\"");
           }
       }

       private static async Task ChunksAsync(string fileName, CommandContext ctx)
       {
           if (string.IsNullOrWhiteSpace(fileName))
           {
               Console.WriteLine("Usage: /chunks <file>");
               return;
           }

           var path = Path.Combine(ctx.NotesFolder, fileName);
           var text = await File.ReadAllTextAsync(path);
           foreach (var chunk in MarkdownChunker.Chunk(fileName, text))
           {
               Console.WriteLine($"[{chunk.HeadingPath}]\n{chunk.Text}\n---");
           }
       }
   }
   ```
   (Notice there is no `continue;` in the handlers — they just finish. `HandleAsync` returning `true` is what tells the loop to skip the agent. The "continue in the wrong place" bug can't happen anymore.)
- [ ] **Replace both `if` blocks in `Program.cs`** (`/search` and `/chunks`) with one call. Placement: inside the `while (true)` loop, right after the empty-input check and before `loop.AskAsync(input)`. Build the context once, before the loop, after `store` and `embeddingGenerator` exist:
   ```csharp
   var ctx = new CommandContext { Store = store, Embeddings = embeddingGenerator, NotesFolder = "samples/notes" };

   // ...inside while (true), after the empty-input check:
   if (await CommandHelper.HandleAsync(input, ctx))
       continue; // this continue is directly in the while loop -- there's no inner loop to confuse it with
   ```
- [ ] **Run `dotnet run`** and check all four: `/search holden` and `/chunks <file>.md` behave as before (and print no `[llm]` line); `/serch holden` prints `Unknown command: /serch` instead of going to the model; `hello there` still reaches the agent.

**Observe**
- Typing `/search` with nothing after it now prints a usage line instead of crashing on a slice or embedding an empty string.

## Exercise 2 — Wire chunking into indexing, and prove the heading-path prefix matters

**Goal:** measure, not assume, the effect of including document structure in the embedded text.

**Do**

- [ ] **Update your Lesson 2 indexing loop** to chunk each file before embedding, instead of embedding the whole file as one chunk — embed `chunk.HeadingPath + "\n\n" + chunk.Text` for each chunk `MarkdownChunker.Chunk(...)` returns, and store the `HeadingPath` alongside the vector in your `VectorStore`. Placement: this is an in-place rewrite of the body of the `foreach` loop you wrote in Lesson 2 Exercise 1 (the one that currently does one `GenerateVectorAsync` + one `store.Add` per file) — you're changing what happens inside that loop, not adding a second loop next to it.
- [ ] **Add a `/reindex` command** that re-runs this whole process (simplest version: just clear the store and redo everything; you can add the change-detection skip-logic back once this works). Placement: with the checkpoint's command helper in place, this is one new `case "/reindex":` in `CommandHelper.HandleAsync` that calls a new private `ReindexAsync(ctx)` method — no change to the `while` loop. For `ReindexAsync` to re-run indexing, the indexing code can't stay inline in `Main`: move it into a method that returns a fresh `VectorStore` (e.g. `Indexer.BuildAsync(embeddings, notesFolder)`), call it from `Main` at startup, and call it again from `ReindexAsync`, assigning the result to `ctx.Store`.
- [ ] Run `/reindex`, then run a handful of `/search` queries against your notes and write down the top result's score for each.
- [ ] **Temporarily remove the prefix** — change the embedded text to just `chunk.Text`, with no heading path — and run `/reindex` again.
- [ ] Re-run the exact same `/search` queries.
- [ ] **Restore the prefix** and `/reindex` once more.

**Observe**
- How much did scores and rankings change without the prefix?

## Exercise 3 — Chunk size sweep

**Goal:** feel the tradeoff between too-small and too-large chunks.

**Do**

- [ ] Pick one of your longer notes. Set `maxChunkChars` to `300` in your `MarkdownChunker.Chunk` call, `/reindex`, and run `/chunks` on that note.
- [ ] Set it to `20000`, `/reindex`, and run `/chunks` again on the same note.
- [ ] Run the same `/search` queries from Exercise 2 at both extremes.

**Observe**
- At the small extreme: does the content you're searching for get fragmented across multiple chunks?
- At the large extreme: do scores become "fuzzy" — high for many unrelated queries because each chunk now covers too much ground?

**Discuss**
- Why split only *between* blocks rather than at a fixed character count?
- What would overlapping chunks (each chunk sharing some text with its neighbor) buy you, and what would it cost?
- How would you chunk a PDF, a table, or a source code file differently than markdown prose?
