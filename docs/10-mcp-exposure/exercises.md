# Lesson 10 — Exercises

## Exercise 1 — Stand up a minimal MCP server

**Goal:** get the protocol plumbing working before porting real tools.

**Do**
1. Using an MCP server SDK/library for .NET, create a minimal server exposing a single trivial tool (e.g. `ping` returning "pong").
2. Configure an MCP-compatible client to connect to it (stdio is the simplest transport to start with).
3. Confirm the client can see the tool and call it successfully.

## Exercise 2 — Port your real tools

**Goal:** expose `search_notes`, `read_note`, `list_notes` without rewriting their logic.

**Do**
1. Extract the actual tool logic (the methods themselves, not the agent-loop wiring) if it isn't already separated from your `AgentLoop`/Program wiring.
2. Register them with the MCP server using the *same* descriptions and parameter schemas you tuned in Lesson 5 — don't rewrite them from scratch.
3. Connect your MCP client and call `search_notes` through it with a real query.

**Observe**
- Did you need to change the tool's actual logic at all, or only how it's registered/exposed?

## Exercise 3 — Deliberately do not expose the write tool

**Goal:** reason through the access-control implications of a remote caller.

**Do**
1. Do *not* register `create_note` with the MCP server.
2. Write down, concretely: if you did expose it, who could call it, from where, and what would need to be true (confirmation UI, authentication, network exposure) before that would be safe.

**Discuss**
- Your console app's confirmation step from Lesson 7 runs in the same terminal the user is sitting at. If `create_note` were called over MCP from a remote client, where would a confirmation prompt even need to live?
- What's genuinely new about the trust boundary here compared to Lesson 7's prompt-injection discussion — is it a new risk, or the same risk with a longer wire between the untrusted content and the action?
- If you later wanted to expose write access safely, what would the minimum safeguard be (e.g. an explicit allow-list, a required human approval step in the MCP flow itself)?
