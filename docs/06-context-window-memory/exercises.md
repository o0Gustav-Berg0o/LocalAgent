# Lesson 6 — Exercises

## Exercise 1 — Overflow the window on purpose

**Goal:** see what actually happens when context runs out.

**Do**
1. Restart the chat server with a small context window (e.g. `-c 2048`).
2. Ask a question that triggers a `search_notes` call returning several chunks.

**Observe**
- What happens? An explicit error, or a silently truncated/degraded response? Check the server log either way.

## Exercise 2 — Watch history grow

**Goal:** connect trace output to actual token cost.

**Do**
1. Restart the chat server with a large context window (e.g. `-c 32768`).
2. Add trace output printing the message count (and, if you can, an estimated token count) before every model call.
3. Ask ten search-requiring questions in a row.

**Observe**
- How does message/token count grow per turn? Does it grow linearly?

## Exercise 3 — Trimming strategies, compared

**Goal:** compare blunt trimming against a smarter alternative.

**Do**
1. Set `MaxHistoryMessages` very low (e.g. 6). Ask a follow-up question that references an answer from several turns back.
2. Set it very high (e.g. 80) and ask ten search-heavy questions in a row until the context window overflows. Predict in advance, from your token budget, roughly which question will trigger the overflow — then check if you were right.
3. Implement the improved strategy: instead of dropping whole old turns, replace old tool-result message content with a short placeholder like `"[old result removed]"`, keeping the surrounding question/answer pairs intact.
4. Repeat the follow-up-question test from step 1 with this new strategy.

**Observe**
- Does the placeholder approach let the model keep the thread of conversation better than hard-dropping whole turns, at a similar token cost?

**Discuss**
- When would summarizing old history (instead of trimming it) be worth the extra complexity and cost?
- At what point does "more context" stop helping, even if you have room for it? (Hint: think about small models and prompt length, not just token limits.)
