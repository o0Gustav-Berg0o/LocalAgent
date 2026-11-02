# Lesson 4 — Exercises

## Exercise 1 — Wire up `search_notes`

**Goal:** retrieval becomes something the model chooses to invoke.

**Do**
1. Expose your Lesson 2 search as a tool: `search_notes(query: string)`, returning formatted hits with source and heading path.
2. Add a system prompt instruction: always call `search_notes` first for any question that might relate to the notes.
3. Ask a question whose answer is in your notes. Check the trace: did it search? With what query?
4. Ask a pure greeting ("Hi, who are you?"). Did it search anyway? Should it have?

## Exercise 2 — Break asymmetric search on purpose

**Goal:** feel what a "quiet" RAG bug looks like.

**Do**
1. Note the ranking and scores for three real questions.
2. Remove the query-side instruction prefix and re-run the same three questions (no reindex needed — only the query embedding changes).
3. Restore the prefix and confirm scores return to their original levels.

**Observe**
- Did rankings change? Did scores change even when the top result stayed the same?

## Exercise 3 — Make "I don't know" a correct answer

**Goal:** separate grounded answers from hallucinated ones.

**Do**
1. Ask about something that is genuinely absent from your notes (a topic you never wrote about).
2. Does the model say it doesn't know, or does it answer anyway, confidently, from training data?
3. Set a `MinScore` threshold so `search_notes` returns "No relevant passages found" when nothing scores above it.
4. Strengthen the system prompt: "Base your answer only on tool results; if they don't contain the answer, say so."
5. Repeat the missing-topic question.
6. Now try the opposite: soften the system prompt (remove the "only use tool results" instruction) and ask again.

**Discuss**
- What's actually worse for a user: a confident wrong answer, or an overly cautious "I don't know" on something the notes did contain?
- A similarity score is not a truth score. What exactly does a high cosine similarity tell you, and what does it *not* tell you?
