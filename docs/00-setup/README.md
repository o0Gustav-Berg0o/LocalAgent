# Lesson 0 — Setup & Orientation

**Slides:** _link added once the deck is created_

## Learning objectives

- Get a local chat model and a local embedding model running as servers.
- Understand why an agent needs *two* models, not one.
- Verify the plumbing works before writing a single line of agent code.

## Why two models?

A chat model is good at understanding and producing language. It is not good at turning text into something you can efficiently *compare* — "is this paragraph about the same thing as that question?" is not a question a chat model answers cheaply at scale.

An **embedding model** solves a narrower problem: turn text into a vector (a list of numbers) such that texts with similar meaning get similar vectors. Comparing two vectors (cosine similarity) is cheap — microseconds even over thousands of chunks.

So the architecture has two local servers:

| Server | Model | Job |
|---|---|---|
| Chat | e.g. `Qwen3-VL-4B-Instruct` | Understands the user, decides which tools to call, writes the final answer |
| Embedding | e.g. `Qwen3-Embedding-0.6B` | Turns text (notes and questions) into vectors for search |

Both run via [llama.cpp](https://github.com/ggml-org/llama.cpp)'s `llama-server`, which exposes an OpenAI-compatible HTTP API. That matters: your C# app never talks to llama.cpp directly, it talks to a generic "OpenAI-shaped" HTTP endpoint. Swap the endpoint later (a cloud provider, a different local server) and nothing else in the app needs to change.

## What you'll do

1. Install the .NET SDK (version 10+) and a CUDA (or CPU) build of `llama-server`.
2. Download a small instruct model with tool-calling support (a 4B-class Qwen3 model is a good default) and a matching embedding model.
3. Start both servers and confirm they respond to a plain HTTP request.
4. Scaffold an empty .NET console project — this is the shell every later lesson builds into.

This lesson produces no agent behavior yet. That's intentional: get the infrastructure boring and reliable first, so every bug in later lessons is actually about the agent, not about a misconfigured server.

## Key terms introduced here

- **`llama-server`** — the process that loads a GGUF model file and serves it over HTTP.
- **GGUF** — a quantized model file format llama.cpp understands.
- **Quantization** (e.g. `Q4_K_L`, `Q8_0`) — compressing model weights to use less memory, at a small quality cost.
- **Context window (`-c`)** — the maximum number of tokens (roughly, word-pieces) the model can see at once. Set when the server starts, not changeable per request.
