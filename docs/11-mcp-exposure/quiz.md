# Lesson 11 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. What problem does MCP solve that "just wire your tools into your own agent loop" doesn't?
2. What changes when you expose a tool over MCP instead of calling it in-process, and what stays the same?
3. Why is this lesson's work described as "the payoff of Lesson 1's design," specifically?
4. Why shouldn't `create_note` be exposed over the MCP server built in this lesson?
5. Besides "what the tool does," what new question do you have to think about once a tool is reachable over MCP?

## Answers

1. Without a shared protocol, every other agent (a teammate's app, Claude Desktop, a different framework) that wants to use your tools would have to reimplement the integration from scratch; MCP standardizes the boundary so any compatible client can connect and use them the same way.
2. The transport changes (a protocol handshake over stdio/HTTP instead of an in-process call), and who's allowed to connect becomes a real question. The tool's actual logic, its description, and its schema don't need to change at all.
3. Because Lesson 1 already cleanly separated "tool definition" from "agent loop" — exposing tools over MCP is only a low-effort change in transport precisely because that separation already existed.
4. Because a remote MCP client calling it wouldn't have the same local, per-action confirmation guarantee from Lesson 7 — exposing a write tool without an equivalent safeguard would be a real, new risk.
5. Who's allowed to connect to the server at all — access and connection control becomes a real concern, not just what the model is permitted to request.
