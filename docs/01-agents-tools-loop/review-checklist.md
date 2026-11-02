# Lesson 1 — Review Checklist

- [ ] You can ask "what's 17 * 23" and get a correct answer that clearly went through your `calculate` tool (visible in trace), not the model's own arithmetic.
- [ ] You can ask something needing no tools at all and the model answers directly, without a spurious tool call.
- [ ] Your loop has a hard iteration cap, and you've verified it actually stops a runaway loop (temporarily remove a tool's correct behavior to simulate confusion, and confirm the cap kicks in).
- [ ] A thrown exception inside a tool does not crash your app — it becomes a tool-result string the model sees.
- [ ] Your trace output shows, for every turn: which tool was called, with what arguments, and what came back.

## Self-check questions

1. In your own words: what's the difference between the model "running" a tool and the model "requesting" a tool call?
2. Why is a `[Description]` on a tool parameter part of the prompt, functionally — not just documentation for humans reading your code?
3. If you removed the iteration cap entirely, what's the worst case that could happen with a small, easily-confused model?
