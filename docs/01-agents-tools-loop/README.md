# Lesson 1 — Agents, Tools & the Loop

**Slides:** https://docs.google.com/presentation/d/1RER_wUZzuF7AGUdPhFTkM3h_ZMW7OUV9QINBiq9cy4Y/edit

## Learning objectives

- Understand that "an agent" is a loop, not a single model call.
- Understand the strict separation between the model *requesting* an action and your code *performing* it.
- Build your first tool and your first (very small) agent loop.

## The librarian analogy

Imagine asking a librarian: *"What do my notes say about reconnecting WiFi?"* The librarian doesn't already know — they have to:

1. Understand the question.
2. Decide to go look something up.
3. Actually look it up.
4. Read what they found.
5. Summarize it back to you, citing where it came from.

Steps 1, 2, and 5 are language tasks — your chat model's job. Step 3 and 4 are *actions on the world* — things the model cannot do by itself. It can't open a file. It can't query a database. It can only produce text.

**A tool bridges that gap.** You give the model a description of an action it can *request* ("search_notes(query: string)"), and your code is the one that actually performs it when asked.

## The loop

```text
User asks a question
    → send conversation history + tool definitions to the model
    → model replies with text, OR a request to call a tool (or several)
    → if it's a tool request: your code runs the tool, gets a result
    → the result is appended to the conversation as a "tool result" message
    → send the updated conversation back to the model
    → repeat until the model answers with plain text (no more tool requests)
```

This is **the** fact to internalize this lesson: *the model never executes anything*. It emits a structured request (tool name + JSON arguments). Your code decides whether to run it, how to run it, and what to tell the model happened. If your tool throws an exception, don't let the loop crash — turn the error into text and feed it back as the tool result, so the model gets a chance to recover (e.g. retry with corrected arguments).

## Why a loop, not a single call?

A single question might require multiple tool calls — search, then read a specific file, then maybe do a calculation. The model can't know in advance how many steps it needs. So you loop, re-sending the growing conversation each time, until the model is satisfied it has enough to answer — or until you hit a safety limit (a max number of iterations), because nothing stops a confused small model from calling the same tool forever.

## What you'll build

- A `get_datetime` tool and a `calculate` tool — deliberately boring, so you can focus on the loop mechanics instead of RAG.
- A hand-written agent loop: send history → detect tool calls → run them → append results → repeat.
- Simple trace output so you can *see* every model request and tool execution as it happens. This is non-negotiable for debugging agents — if you can't see what the model asked for, you're debugging blind.
