# Lesson 6 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. In what sense does the model have "no memory between calls"?
2. What sets the hard limit on how long a conversation can get, and when is that decided?
3. Why are tool results, specifically, called out as the most expensive and most disposable part of history?
4. What's the main weakness of trimming history by message count rather than by token count?
5. What does the "improved" trimming strategy do differently from simply dropping old messages?

## Answers

1. Every call to the chat server is independent; the model doesn't remember anything from before. Your code has to resend the entire conversation — every message, tool call, and tool result — every single turn for it to appear to "remember."
2. The context window (`-c`), set when the chat server starts. It can't be renegotiated mid-conversation.
3. A single tool result (e.g. several retrieved chunks) can cost far more tokens than the question that triggered it, and it's also the part of history most likely to go stale and become useless once a few turns have passed.
4. It has no idea how many actual tokens those messages cost, so it can trim too conservatively (wasting usable context) or too late (overflowing the window), depending on how token-heavy the messages actually are.
5. It replaces old tool results with a short placeholder instead of deleting the whole turn, keeping the conversational thread (the question and the model's reply) intact while still freeing most of the tokens.
