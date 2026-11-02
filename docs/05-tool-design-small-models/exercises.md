# Lesson 5 — Exercises

## Exercise 1 — Starve the description

**Goal:** see the effect of a vague tool description directly.

**Do**
1. Change `search_notes`'s description to just `"Search"`, and its `query` parameter's description to `"q"`.
2. Ask three questions whose answers are clearly in your notes.

**Observe**
- Is the tool still called? With sensible arguments, or does the model hesitate / call it with odd arguments?

## Exercise 2 — Add overlapping, confusing tools

**Goal:** see what happens when the model has to choose between similar options.

**Do**
1. Restore a good description on `search_notes`.
2. Add three more tools that all call the *same* underlying search function, named `find_documents`, `lookup_info`, and `query_knowledge_base`, each with a slightly different but overlapping description.
3. Ask the same three questions from Exercise 1.

**Observe**
- Which tool does the model pick? Does it sometimes call more than one, or the wrong one?

## Exercise 3 — Remove the system prompt's directive

**Goal:** isolate the effect of instruction placement.

**Do**
1. Remove the three extra tools, keep only `search_notes` with a good description.
2. Remove the system prompt instruction telling the model to search first.
3. Ask the same questions again.

**Observe**
- Does the model skip searching more often now, even with a good tool description?

**Discuss**
- Why is "few, clear tools" a hard design constraint for small models rather than a style preference?
- How would you expect an 8B or 14B model, or a large cloud model, to behave differently on these same exercises?
- If you genuinely needed 15 tools in a real application, how would you structure them to stay usable by a small model (e.g. grouping, routing, or a larger model for tool selection)?
