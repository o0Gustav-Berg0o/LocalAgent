# Lesson 8 — Measuring Quality: an Eval Harness

**Slides:** https://docs.google.com/presentation/d/1R4dREpngqfLWuMU7pCbGNuQRuK7Fy3PqktUiXuQPNGc/edit
**Quiz:** [quiz.md](quiz.md) · https://docs.google.com/document/d/1Cz0v0LNCX71PJ_zAJZS9EUL16LtgRaBV0NngQgvfDww/edit

## Learning objectives

- Stop judging retrieval quality by vibes and start measuring it.
- Build a small labeled evaluation set and compute recall@k.
- Use the harness to make an actual, evidence-backed decision about a chunking or retrieval parameter.

## Why "it feels right" isn't evaluation

By this point you've eyeballed a lot of `/search` output and formed opinions — "chunk size 2000 feels better than 300." That intuition is useful for debugging but worthless for comparing two configurations rigorously, because you tested different settings on different days with different queries and no fixed ground truth. Evaluation means fixing the questions and the correct answers *first*, then running every configuration against the exact same fixed test, so differences in score are attributable to the thing you changed and nothing else.

## recall@k

For a given question, you know (because you wrote the test) which source file(s) actually contain the answer. **recall@k** asks: among the top-k search results, did at least one of the expected sources appear? Average that over all your test questions and you get a single number you can compare across configurations.

This is deliberately simple — it doesn't check whether the model's final *answer* was correct, only whether retrieval surfaced the right material. That's intentional: it isolates retrieval quality from everything the chat model might do with it, which is exactly the layer most of this course's RAG lessons have been tuning.

## What you'll build

- A fixed set of ~15 questions, each with its expected source file (or files) — written by you, based on your own notes, so you can judge correctness.
- A small script/command that runs every test question through `/search` (not the full chat loop — keep this isolated like Lesson 2) at a given `k`, checks whether an expected source appears in the top-k, and reports overall recall@k.
- At least one real before/after comparison: run the harness, change one retrieval parameter (chunk size, `TopK`, with/without heading prefix, with/without query instruction), run it again, and report the actual delta.
