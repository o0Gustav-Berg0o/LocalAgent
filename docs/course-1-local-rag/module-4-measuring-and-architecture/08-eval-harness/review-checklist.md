# Lesson 8 — Review Checklist

- [ ] You have a fixed, written-down set of ~15 questions with expected sources, including a few "no correct answer" cases.
- [ ] Your harness computes recall@k without involving the chat model at all.
- [ ] You ran a real before/after comparison on at least one retrieval parameter and can state the exact recall@k numbers on both sides.
- [ ] You can explain at least one case where your eval set's result disagreed with your earlier intuition (or confirmed it, which is also worth noting).
- [ ] Your harness correctly handles "no correct source exists" questions rather than treating them as automatic failures or automatic passes.

## Self-check questions

1. What does recall@k measure, and what does it deliberately *not* measure?
2. Why does this harness call search directly instead of running full questions through the chat agent?
3. If recall@6 is high but users still complain about bad answers, what would you check next, given what this harness does and doesn't cover?
