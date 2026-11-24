# Lesson 4 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why expose search as a tool the model calls, instead of always running it and injecting results into every prompt?
2. What's the practical risk of exposing retrieval only as an optional tool call rather than always-on?
3. Define grounding and hallucination, in your own words.
4. What does a `MinScore` cutoff let the agent do correctly that it couldn't do without one?
5. What's the actual symptom of breaking asymmetric search (removing the query-side instruction prefix)? Does it throw an error?

## Answers

1. Auto-injecting search results on every turn wastes tokens on turns that don't need it (a greeting, a calculation) and adds noise; exposing it as a tool lets the model decide if and when retrieval is actually needed.
2. A small model might skip calling it when it should have, or call it with a poor query — mitigated by an explicit system-prompt instruction to search first for relevant questions.
3. Grounding: the answer is actually built from retrieved text and cites where it came from. Hallucination: the answer comes from the model's own training data instead, sounds equally plausible, and is indistinguishable from a grounded answer just by reading it.
4. It lets `search_notes` honestly report "no relevant passages found" when nothing in the notes is actually relevant — which is what makes an "I don't know" answer the *correct* response, instead of the model being handed weak or irrelevant chunks and forced to answer from them anyway.
5. No error at all — search quietly gets worse. Scores and rankings degrade silently, with nothing visibly broken.
