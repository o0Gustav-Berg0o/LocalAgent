# Lesson 3 — Chunking Strategies

**Slides:** https://docs.google.com/presentation/d/1TAU4u_FTCMj1CStEA67WmSg__PJQaklQHJS_xc4fQbE/edit

## Learning objectives

- Understand why documents must be split before embedding, and what breaks if you split badly.
- Implement a structure-aware (heading-based) chunker.
- See directly how chunk size and context prefixing change search quality.

## Why chunk at all?

An embedding model compresses a text into one fixed-size vector, no matter how long the text is. Embed an entire 5,000-word document as one vector, and that vector represents an *average* of everything in it — too vague to match a specific question about one paragraph. Embed too small a fragment (a single sentence with no surrounding context), and the vector may be too narrow to be found by a differently-worded question, or ambiguous on its own ("return it within 30 days" — return *what*?).

A **chunk** is the unit you actually embed and retrieve. Getting chunk boundaries right is one of the highest-leverage decisions in a RAG system — more impactful, usually, than which embedding model you pick.

## Structure-aware chunking

The simplest chunker just cuts text every N characters. This routinely slices a sentence, a table row, or a code block in half. A better approach parses the document's actual structure (headings, paragraphs, lists, code blocks) and only ever splits *between* blocks, accumulating blocks into a chunk until a size budget is hit.

**Heading path as context.** A paragraph under "## Reconnection" doesn't say what should reconnect — the heading above it does. Prefixing each chunk's embedded text with its heading path (`file.md > WiFi > Reconnection`) folds that context into the vector itself. This is one of the cheapest, highest-impact improvements available in RAG, and you'll measure it directly in this lesson's exercises.

## What you'll build

- A markdown-aware chunker: parse the document into blocks, accumulate blocks into a chunk until a character budget is hit, and never split mid-block (a code block is never cut in half).
- Heading-path tracking: as you encounter headings, update a running path and prefix every chunk's embedded text with it.
- An updated `/chunks <file>` command showing exactly how a given file gets split, so you can inspect chunking without re-running search.
