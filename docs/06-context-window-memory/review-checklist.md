# Lesson 6 — Review Checklist

- [ ] You've seen, firsthand, what happens when a request exceeds the server's context window.
- [ ] Your trace output shows growing message/token counts turn by turn, and you can explain the growth rate.
- [ ] You implemented message-count-based trimming and verified it actually drops the oldest turns once the limit is hit.
- [ ] You implemented the "replace old tool results with a placeholder" improvement and compared it against hard trimming on a real follow-up question.
- [ ] You can roughly estimate, before running it, how many search-heavy turns your current `-c` and `MaxHistoryMessages` settings will support.

## Self-check questions

1. Why is "the model remembers the conversation" a useful simplification but a technically wrong description of what's happening?
2. Which single type of message in history costs the most tokens, and why does that make it the best trimming target?
3. If you doubled `MaxHistoryMessages` without checking your token budget, what's the realistic failure mode mid-conversation?
