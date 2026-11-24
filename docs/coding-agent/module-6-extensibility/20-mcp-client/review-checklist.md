# Module 6, Lesson 20 — Review Checklist

## Configuration

- [ ] Servers are configured in one file the user writes, with a name, a program, an argument list, and environment variable names.
- [ ] Two servers with the same name are refused, and a program with a path is refused.
- [ ] Environment values are copied from the client's environment, never written in the file.

## Connection

- [ ] The client waits for the initialise reply before asking for tools.
- [ ] Each tool name is prefixed with the server name, and the prefix is shown.
- [ ] I checked the MCP client library's method names against the protocol version I target.

## Permissions

- [ ] External tools go through the same permission check, with the prefixed name as the target.
- [ ] A server's statement about its own safety does not change the rule.

## Descriptions and results

- [ ] Descriptions are shown at connection, cut to a set length.
- [ ] Phrases that address the model produce a warning that names the phrase.
- [ ] No description or result can change a rule, a mode, or a setting. Tested with a poisoned result.

## Limits and lifecycle

- [ ] Each external call has a timeout, and a timeout returns a failure, not a hang.
- [ ] Results are limited in size, and cuts are stated.
- [ ] Servers start on demand, stop after a period without calls, and stop at session end, with their process trees.

## Reflection

- [ ] I can say which external tool I would trust least, and what I would change in its configuration.
- [ ] I can explain what the client protects against, and what it cannot protect against.
