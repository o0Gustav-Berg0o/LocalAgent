# Lesson 3 — Review Checklist

- [ ] `/chunks <file>` shows each chunk with its heading path, and no chunk cuts a code block or table in half.
- [ ] A heading with no body text under it produces zero chunks.
- [ ] YAML frontmatter never appears inside a chunk's text.
- [ ] You measured, with actual scores, how much removing the heading-path prefix hurts search quality.
- [ ] You observed concrete degradation at both chunk-size extremes (too small, too large), not just in theory.

## Self-check questions

1. Why does chunking quality usually matter more than embedding model choice for a given RAG system?
2. Describe, in your own words, what goes wrong with fixed-size character chunking that structure-aware chunking avoids.
3. If you had to add chunk overlap, where exactly would you add it in your chunker's logic?
