# Lesson 11 — Review Checklist (Course-Wide)

Use this as the final gate before considering the course complete.

- [ ] Your agent loop is hand-written and you can explain every step of it without looking at your own code.
- [ ] Chunking is structure-aware and includes heading-path context; you measured the effect of removing it.
- [ ] Retrieval is a tool the model chooses to call, with a working grounding/"I don't know" path, verified with a genuinely absent topic.
- [ ] Tool descriptions were deliberately tuned, and you've seen them fail when vague.
- [ ] History trimming is implemented and you understand your current token budget at your current `-c`.
- [ ] A prompt injection succeeded against your agent at least once, on purpose, and you can describe why a stronger system prompt alone didn't fully stop it.
- [ ] `create_note` only ever writes after an explicit, per-action, content-visible confirmation.
- [ ] You have a labeled eval set and have used recall@k to make at least one real, evidence-backed decision.
- [ ] Hybrid search is implemented and measured against vector-only search on identifier-style queries.
- [ ] Your tools are callable through MCP by an external client, and `create_note` is deliberately not exposed that way.
- [ ] You completed the full demo run in Exercise 1 and have the trace output to show it.
- [ ] You wrote an honest retrospective — including what's still weak, not just what works.

## Final self-check

If someone unfamiliar with this course asked you "why not just use LangChain / a managed RAG service for this," could you give a specific, technical answer grounded in what you built — not just "it's more educational"?
