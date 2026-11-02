# Lesson 0 — Exercises

## Exercise 1 — Start the model servers

**Goal:** have two `llama-server` processes running and reachable.

**Do**
1. Download a GGUF chat model with tool-calling support and a GGUF embedding model.
2. Start the chat server with `--jinja` (required for tool calls) on port 8080.
3. Start the embedding server with `--embedding --pooling last` on port 8081 (check the model card — some embedding models need `mean` or `cls` pooling instead).
4. Open `http://localhost:8080` in a browser (llama.cpp's built-in UI should load) and `curl http://localhost:8081/health`.

**Observe**
- What does the server log print about `n_ctx_slot`? Does it match the `-c` value you passed?
- How much VRAM does each server report using (`nvidia-smi`)?

**Discuss**
- Why does the embedding server need a much smaller context window than the chat server?

## Exercise 2 — Talk to the chat server directly

**Goal:** confirm the OpenAI-compatible API works before any C# code exists.

**Do**
1. Send a raw HTTP POST to `http://localhost:8080/v1/chat/completions` with a simple user message (curl, Postman, whatever you like).
2. Send a raw HTTP POST to `http://localhost:8081/v1/embeddings` with a short string.

**Observe**
- How long is the returned embedding vector? (This number — the embedding dimension — matters later.)

## Exercise 3 — Scaffold the project

**Goal:** have an empty but runnable .NET console app.

**Do**
1. Create a new .NET console project.
2. Add the `Microsoft.Extensions.AI` and `Microsoft.Extensions.AI.OpenAI` packages (these give you `IChatClient` / `IEmbeddingGenerator` abstractions over the OpenAI-compatible API).
3. Write a `Main` that connects to the chat server and prints the model's reply to a hardcoded "Hello" message. No tools, no RAG — just confirm the wiring.
