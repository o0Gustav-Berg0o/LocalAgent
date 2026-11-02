# Lesson 3 — Exercises

## Exercise 1 — Structure-aware chunker

**Goal:** split documents without ever cutting a block in half.

**Do**
1. Parse your notes into blocks using a markdown parser (e.g. Markdig for .NET).
2. Track a running "heading path" as headings are encountered (reset the buffer, update the path, continue).
3. Accumulate non-heading blocks into a buffer; when adding the next block would exceed a `MaxChunkChars` budget, flush the buffer as one chunk first.
4. Skip YAML frontmatter. A heading with no content under it produces no chunk.
5. Add a `/chunks <file>` command printing each resulting chunk with its heading path.

## Exercise 2 — Prove the heading-path prefix matters

**Goal:** measure, not assume, the effect of including document structure in the embedded text.

**Do**
1. Run a handful of `/search` queries against your notes and note the top result's score for each.
2. Temporarily embed chunks as *just the raw text*, without the heading-path prefix. Re-index.
3. Re-run the same `/search` queries.
4. Restore the heading-path prefix and re-index again.

**Observe**
- How much did scores and rankings change without the prefix?

## Exercise 3 — Chunk size sweep

**Goal:** feel the tradeoff between too-small and too-large chunks.

**Do**
1. Set `MaxChunkChars` very small (e.g. 300), reindex, and inspect `/chunks` on a longer note.
2. Set it very large (e.g. 20000), reindex, and inspect again.
3. Run the same `/search` queries at both extremes.

**Observe**
- At the small extreme: does context get fragmented across multiple chunks?
- At the large extreme: do scores become "fuzzy" — high for many unrelated queries because each chunk covers too much ground?

**Discuss**
- Why split only *between* blocks rather than at a fixed character count?
- What would overlapping chunks (each chunk sharing some text with its neighbor) buy you, and what would it cost?
- How would you chunk a PDF, a table, or a source code file differently than markdown prose?
