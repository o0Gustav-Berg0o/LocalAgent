# Module 6, Lesson 20 — Quiz

Write your own answer to each question first, then check the answer key.

1. What does the MCP client do, in four steps?
2. Why are external tools prefixed with the server's name?
3. A server says in its description that its tool is safe and read-only. What should the client do with that statement?
4. What is tool poisoning, and where does the malicious text sit?
5. Why must a server's result be unable to change a permission rule?
6. Why does a server run with the user's permissions, and what does that mean for the client's protection?

---

## Answer key

1. It starts or connects to a server, lists the server's tools, passes them to the model, and forwards the model's calls to the server and returns the results.
2. So that two servers can have tools with the same name without clashing, and so that the user sees the source in every trace line and in every permission rule.
3. The client should treat it as the server's claim, not as a rule. The permission rules decide what the tool may do, whatever the description says.
4. Tool poisoning is a tool description written to change the model's behaviour, for example to skip confirmations. The text sits in the tool list the model reads, not in a file.
5. A result is data from an external process. If it could change a rule, any server could grant itself any permission, and the user's control would be gone.
6. The server is a process on the user's machine, and it can read and write what the user can. The client cannot see or control its internal actions, so the protection is the choice of server, the rules on what the client may call, and the timeouts and limits on the calls.

---

## Scoring

- 5 or 6 correct: ready for Lesson 21.
- 3 or 4 correct: re-read sections 3.2, 3.4, and 3.5 of the README, then repeat Exercises 3 and 4.
- Fewer than 3: repeat Exercises 1 to 4.
