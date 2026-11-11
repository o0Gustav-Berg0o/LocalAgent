# Lesson 7 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why can't a language model architecturally distinguish "trusted instruction" from "untrusted content"?
2. Why is a stronger system prompt not a sufficient defense against prompt injection on its own?
3. What's the actual, reliable mitigation this lesson teaches, instead of just a better prompt?
4. Why must write confirmation happen per-action, rather than once per session (a "write mode: on" toggle)?
5. What's the risk difference between a read-only tool and a tool with a side effect, under a prompt injection attack?

## Answers

1. It processes all text it sees the same way — there's no hard structural wall between an instruction and content, the way there is between, say, SQL code and a SQL parameter.
2. It genuinely helps but isn't a guarantee, especially against a model this size, which has less capacity to resist text that looks authoritative.
3. Limiting what the agent can *do*, not just what it's told — any tool with a side effect requires explicit human confirmation of that specific action, every single time, architecturally, not just a prompt asking it to behave.
4. A one-time toggle would let an injected instruction slip a write through later in the same session without the user ever seeing the specific content being written; per-action confirmation means the user always sees exactly what's about to be written, every time.
5. A read-only tool's worst case is the model reading and maybe repeating back something malicious, which you can catch by inspecting output. A tool with a side effect could actually *do* something — write, send, delete — if not gated, which is a fundamentally bigger risk.
