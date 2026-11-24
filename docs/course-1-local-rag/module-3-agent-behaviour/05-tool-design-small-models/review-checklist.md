# Lesson 5 — Review Checklist

- [ ] You directly observed worse tool-calling behavior with a vague `search_notes` description.
- [ ] You directly observed confusion (wrong tool, or multiple redundant calls) when overlapping tools were added.
- [ ] You directly observed the model skipping search more often once the system-prompt directive was removed.
- [ ] Your final tool set is back to one well-described `search_notes`, with a system prompt that tells the model when to use it.
- [ ] You can point to the exact description text responsible for the model's correct behavior, not just "it works now."

## Self-check questions

1. What's the practical difference, from the model's point of view, between a tool's JSON schema and a sentence in the system prompt?
2. Why would adding more tools "just in case" quietly make your existing tools less reliable?
3. Name one tool-design mistake you made in Lesson 1 or 4 without realizing it, and how you'd fix it now.
