# Lesson 4 — Review Checklist

- [ ] The model calls `search_notes` before answering questions about your notes, visible in trace.
- [ ] The model does *not* call `search_notes` on a plain greeting.
- [ ] You measured a concrete score/ranking difference between correct and broken asymmetric query embedding.
- [ ] Asking about a genuinely absent topic produces an explicit "I don't know" / "not found in notes" response, not a confident guess.
- [ ] You've seen the opposite failure too: with a softened system prompt, the same missing-topic question gets answered anyway from training data.
- [ ] Citations in the model's answer (`file > heading`) actually match what appeared in the tool result in trace — not fabricated ones.

## Self-check questions

1. What is the actual risk of exposing retrieval as a tool rather than always injecting it — and why is that risk acceptable for this course's use case?
2. Why is "the model cited a source" not, by itself, proof that the answer is grounded?
3. If you raised `MinScore` too high, what's the likely user-visible failure mode?
