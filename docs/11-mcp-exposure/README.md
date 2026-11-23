# Lesson 11 — Exposing Tools via MCP

**Slides:** https://docs.google.com/presentation/d/15HAkKKqWOd1PmhS53ZkE4QxoNSivuXgPcJPocv1CcI4/edit
**Quiz:** [quiz.md](quiz.md) · https://docs.google.com/document/d/14hDFcpgkevI3VTck1zL856ccbQJB0dsyzfrtnPIvvZA/edit

## Learning objectives

- Understand what MCP (Model Context Protocol) actually standardizes, and why.
- Expose your existing tools (`search_notes`, `read_note`, `list_notes`) as an MCP server.
- See another agent (not your own console app) use your tools.

## What MCP standardizes

Every lesson so far has had *your* C# code define tools and wire them directly into *your* agent loop. That's fine for one app, but it doesn't scale: if you wanted Claude, or a different agent, or a teammate's app to use your note-search capability, they'd each need to reimplement the integration from scratch.

**MCP (Model Context Protocol)** standardizes the boundary between "a set of tools/resources" and "an agent that uses them," over a well-defined protocol (JSON-RPC based, typically over stdio or HTTP). Instead of hand-wiring tools into one agent's loop, you expose them once, as an **MCP server**, and *any* MCP-compatible client (Claude Desktop, Claude Code, other agent frameworks) can connect to it and get your tool definitions and call them the same way your own agent loop already does internally.

This is the payoff of everything from Lesson 1: because you already separated "tool definition" from "agent loop" cleanly, exposing the tools over MCP instead of calling them in-process is a change in how calls are *transported*, not a change in what the tools *are* or do.

## What stays the same, what changes

| Stays the same | Changes |
|---|---|
| Tool logic (search, read, list) | How a caller discovers and invokes the tool (protocol handshake instead of in-process call) |
| Tool descriptions and schemas (still prompts — Lesson 5 still applies) | The caller is now a separate process, possibly a separate machine |
| Validation inside each tool (still untrusted input) | You now need to think about who's allowed to connect, not just what the model can request |

## What you'll build

- An MCP server process exposing `search_notes`, `read_note`, and `list_notes` (read-only tools only — do **not** expose `create_note` over MCP without the same per-action confirmation guarantee from Lesson 7, and think hard about whether a remote caller should ever get write access at all).
- A connection from an MCP-compatible client (e.g. Claude Desktop or Claude Code configured with your server) proving a *different* agent can now search your notes.
- A short comparison write-up: what was reused unchanged from Lesson 4, and what had to change for the MCP transport.
