# Lesson 0 — Quiz

Write your own answer to each question first, then check it against the answer key below. Don't peek first — the point is to catch what you don't actually know yet.

## Questions

1. Why does this course use two separate local model servers instead of one?
2. What does the `-c` flag control on `llama-server`, and when can you change it?
3. What is quantization, and what's the tradeoff it makes?
4. Why does your C# app talk to an "OpenAI-compatible HTTP API" instead of talking to llama.cpp directly?
5. What is a GGUF file?

## Answers

1. A chat model is good at understanding and producing language, but not at cheaply comparing meaning across lots of text. An embedding model solves that narrower problem by turning text into vectors that can be compared with cosine similarity in microseconds — two different jobs, two different models.
2. The context window — the maximum number of tokens the model can see at once. It's set when the server starts and can't change per request; you have to restart the server with a new value.
3. Compressing a model's weights (e.g. to `Q4_K_L`, `Q8_0`) so it uses less memory and runs faster, at a small cost to output quality.
4. So that swapping the backend later (a cloud provider, a different local server) only means changing an endpoint URL — your app's code never has to change.
5. A quantized model file format that llama.cpp understands — the actual file on disk holding the model's weights.
