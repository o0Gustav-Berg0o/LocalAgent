# Lesson 0 — Review Checklist

Check each of these yourself before moving on. Don't skip to Lesson 1 until all of these are true.

- [ ] `http://localhost:8081/health` returns a healthy response.
- [ ] A raw HTTP call to the chat server's `/v1/chat/completions` returns a sensible reply.
- [ ] A raw HTTP call to the embedding server's `/v1/embeddings` returns a vector, and you know its dimension (write it down — you'll need it in Lesson 2).
- [ ] Your console app connects to the chat server via `IChatClient` and prints a reply, with zero hardcoded HTTP code of your own.
- [ ] You can explain, in one sentence each, what the chat model and the embedding model are each responsible for.
- [ ] You know where your `-c` (context size), `-ngl` (GPU layers), and `--pooling` flags are set, and why each exists.

## Self-check questions

1. If you stop the embedding server but leave the chat server running, which operations in your (future) app would still work?
2. Why is the OpenAI-compatible HTTP API a better integration point than a native llama.cpp binding, for this course's purposes?
