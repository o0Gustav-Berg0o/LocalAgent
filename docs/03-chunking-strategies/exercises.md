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
           var document = Markdig.Markdown.Parse(text);
           var chunks = new List<Chunk2>();
           var headingStack = new List<(int Level, string Text)>();
           var buffer = new System.Text.StringBuilder();

           void Flush()
           {
               if (buffer.Length == 0) return;
               var path = string.Join(" > ", new[] { fileName }.Concat(headingStack.Select(h => h.Text)));
               chunks.Add(new Chunk2(path, buffer.ToString().Trim()));
               buffer.Clear();
           }

           foreach (var block in document)
           {
               if (block is Markdig.Extensions.Yaml.YamlFrontMatterBlock) continue;

               if (block is Markdig.Syntax.HeadingBlock heading)
               {
                   Flush();
                   headingStack.RemoveAll(h => h.Level >= heading.Level);
                   var headingText = text[heading.Span.Start..(heading.Span.End + 1)].TrimStart('#', ' ');
                   headingStack.Add((heading.Level, headingText));
                   continue;
               }

               var blockText = text[block.Span.Start..(block.Span.End + 1)];
               if (buffer.Length + blockText.Length > maxChunkChars) Flush();
               buffer.AppendLine(blockText);
           }
           Flush();
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
           Console.WriteLine($"[{chunk.HeadingPath}]\n{chunk.Text}\n---");
       continue;
   }
   ```
- [ ] **Run `dotnet run`** and try `/chunks <one of your note filenames>.md`.

**Observe**
- Does any chunk cut a code block, list, or table in half? (It shouldn't — if it does, check your block-span handling.)

## Exercise 2 — Wire chunking into indexing, and prove the heading-path prefix matters

**Goal:** measure, not assume, the effect of including document structure in the embedded text.

**Do**

- [ ] **Update your Lesson 2 indexing loop** to chunk each file before embedding, instead of embedding the whole file as one chunk — embed `chunk.HeadingPath + "\n\n" + chunk.Text` for each chunk `MarkdownChunker.Chunk(...)` returns, and store the `HeadingPath` alongside the vector in your `VectorStore`. Placement: this is an in-place rewrite of the body of the `foreach` loop you wrote in Lesson 2 Exercise 1 (the one that currently does one `GenerateVectorAsync` + one `store.Add` per file) — you're changing what happens inside that loop, not adding a second loop next to it.
- [ ] **Add a `/reindex` command** that re-runs this whole process (simplest version: just clear the store and redo everything; you can add the change-detection skip-logic back once this works).
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
