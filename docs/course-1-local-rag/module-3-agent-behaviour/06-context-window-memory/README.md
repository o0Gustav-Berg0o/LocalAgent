# Lesson 6 — Context Window & Memory

**Slides:** https://docs.google.com/presentation/d/1jL-P7t947eaSrwKhu5WI4c3xvkzcKPWPUfJkou77E7A/edit
**Quiz:** [quiz.md](quiz.md) · https://docs.google.com/document/d/1tP7sfLp5g02eA5DfOFfHkiIfbo9DdTx2DGDCA18j5Nk/edit

## Learning objectives

- Treat the context window as a finite, shared, budgeted resource.
- Understand why the model has no memory between calls — the conversation history *is* its memory.
- Implement history trimming and estimate your token budget.

## The model is stateless

Every call to the chat server is independent. The model doesn't "remember" your previous question — your code resends the *entire* conversation (every user message, assistant reply, tool call, and tool result) on every single turn. What looks like memory is really you re-presenting the whole transcript each time, like handing someone a growing cheat sheet instead of relying on them to recall a conversation.

This has a direct consequence: **the context window (`-c` on the chat server) is the hard limit on how long a conversation can get**, and it's set when the server starts, not something your app can renegotiate mid-conversation.

## Where tokens actually go

A rough budget for a 32K context window, one typical RAG turn:

| Part | Approx. tokens |
|---|---|
| System prompt | ~250 |
| Tool schemas (5 tools) | ~500 |
| One `search_notes` result (6 chunks × ~2000 chars) | ~3,900 |
| Conversation history (grows every turn) | grows |
| Room for the model's answer | ~1,000+ |

Notice how expensive a single tool result is compared to the question itself. Tool results are also the part of history most likely to be stale and least useful to keep around once a few turns have passed — which is exactly why a smarter trimming strategy (Exercise 3 below) targets them specifically.

## Trimming strategy

The simplest approach: count *messages*, not tokens, and drop the oldest whole turns once history exceeds a message-count limit. This is easy to implement and easy to reason about, but it's blunt — it has no idea how many actual tokens those messages cost, so it can either trim too conservatively (wasting context) or too late (overflowing it), depending on how chunky your tool results are.

## What you'll build

- A message-count-based `TrimHistory` that drops oldest turns once a `MaxHistoryMessages` limit is exceeded.
- Trace output showing message count and an estimated token count at every model call.
- (Exercise-only) An improved trim strategy that replaces old tool results with a placeholder instead of deleting whole turns, preserving conversational thread while still freeing most of the tokens.
