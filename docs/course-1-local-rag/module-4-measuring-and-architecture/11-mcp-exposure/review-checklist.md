# Lesson 11 — Review Checklist

- [ ] A minimal MCP server you built responds correctly to a connected client.
- [ ] `search_notes` are callable through MCP by a different agent/client than your own console app.
- [ ] The tool descriptions/schemas used over MCP are the same ones you tuned in Lesson 5, reused rather than rewritten.
- [ ] `create_note` is explicitly *not* exposed over MCP, and you can explain exactly why.
- [ ] You have a written answer for where a write-confirmation step would need to live if you ever did expose a write tool remotely.

## Self-check questions

1. What does MCP actually standardize — the tool's behavior, or the way a caller discovers and invokes it?
2. Why did porting your tools to MCP require little or no change to the tool logic itself, if Lessons 1–7 were done properly?
3. Is exposing a read-only tool over MCP to an untrusted or semi-trusted caller ever risk-free? What's the worst case, even for `search_notes`?
