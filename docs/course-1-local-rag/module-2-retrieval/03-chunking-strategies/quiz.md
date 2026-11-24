# Lesson 3 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why is embedding an entire 5,000-word document as a single vector a bad idea?
2. What's wrong with a chunker that simply cuts text every N characters?
3. What is the "heading path," and why is prefixing it onto a chunk's embedded text valuable?
4. What should happen to a heading in the source file that has no content underneath it?
5. Name one real tradeoff of making chunks very large versus very small.

## Answers

1. The vector ends up representing an average of everything in the document — too vague to match a specific question about just one part of it.
2. It routinely slices a sentence, a table row, or a code block in half, destroying structure and meaning at an arbitrary boundary.
3. It's the chain of headings above a chunk (e.g. `file.md > WiFi > Reconnection`). Prefixing it folds in context a paragraph alone wouldn't carry (what does "it" refer to?), at very low cost for a high search-quality payoff.
4. It should produce no chunk at all — an empty chunk would be meaningless and shouldn't be embedded or stored.
5. Very small chunks fragment related content across multiple chunks and lose surrounding context; very large chunks make scores "fuzzy" — high for many unrelated queries, because each chunk now covers too much ground.
