# Lesson 0 — Exercises

Platform notes below assume Windows + PowerShell, since that matches the reference setup, but every command has an obvious Linux/macOS/bash equivalent (just drop `.exe`, use `curl` the same way, and use `export` instead of `$env:`).

## Exercise 1 — Install llama.cpp and get the models

**Goal:** have `llama-server` installed and two GGUF model files on disk before touching any code.

**Do**

- [ ] **Install llama.cpp.** Go to `https://github.com/ggml-org/llama.cpp/releases` and download a prebuilt package for your platform:
   - NVIDIA GPU: a file with `cuda` in its name, matching your installed CUDA version (check with `nvidia-smi` at the top of its output).
   - No GPU / unsure: a plain CPU build (e.g. containing `bin-win-x64` or similar, no `cuda`/`vulkan`/`hip` in the name).
   Extract the zip to a folder you'll remember, e.g. `C:\llama.cpp\`. Inside you'll find `llama-server.exe`.
- [ ] **Make `llama-server` easy to run.** Either add `C:\llama.cpp\` to your `PATH` (System Properties → Environment Variables → Path → New), or just remember to `cd C:\llama.cpp` before running it, or always invoke it by full path (`C:\llama.cpp\llama-server.exe`). Verify it works: open PowerShell and run
   ```powershell
   llama-server.exe --version
   ```
   If that prints a version string, it's installed correctly.
- [ ] **Create a folder for model files**, e.g. `C:\models\`. You'll download two `.gguf` files into it: one chat model, one embedding model.
- [ ] **Get the chat model.** You need a small (3–4B parameter) instruct model with tool-calling support. The reference setup uses `Qwen3-VL-4B-Instruct` quantized to `Q4_K_L`. Two ways to get it:
   - **Easiest — let llama.cpp download it for you.** `llama-server` can pull directly from Hugging Face with the `-hf` flag, in the form `-hf <org>/<repo>:<QUANT>`. Search `https://huggingface.co/models?search=Qwen3-VL-4B-Instruct%20GGUF` for a GGUF-quantized repo of this model, open it, and copy the repo id shown at the top of the page (e.g. `SomeOrg/Qwen3-VL-4B-Instruct-GGUF`). You'll use that repo id directly in the server command in Exercise 2 — the file itself downloads automatically on first run, into llama.cpp's local model cache.
   - **Manual.** On the same Hugging Face repo page, open the **Files and versions** tab, find the file whose name ends in `-Q4_K_L.gguf` (or whichever quantization you chose), and download it into `C:\models\`.
- [ ] **Get the embedding model** the same way: search for `Qwen3-Embedding-0.6B GGUF`, and get the `Q8_0` quantization, either via `-hf` or by downloading the file into `C:\models\`.
- [ ] **Note which quantization you used for each model** — you'll need to recognize it later if search quality looks off after you change it.

**Observe**
- How large is each downloaded `.gguf` file? Roughly how does that compare to the number in the model's name (e.g. "4B" parameters)?

**Discuss**
- Why would a 4B-parameter model's file be a few gigabytes rather than, say, a few megabytes?

## Exercise 2 — Start the model servers

**Goal:** have two `llama-server` processes running and reachable at the same time.

**Do**

- [ ] **Open two separate PowerShell windows** — each `llama-server` process keeps running and occupies its terminal, so each model needs its own window that you leave open for the rest of this course.
- [ ] **In the first window**, start the chat server. If you downloaded the file manually, point `-m` at its exact path; if you're using the Hugging Face puller, replace `-m <path>` with `-hf <repo>:<quant>` instead:
   ```powershell
   llama-server.exe -m C:\models\Qwen3-VL-4B-Instruct-Q4_K_L.gguf -c 32768 -np 1 -ngl 99 --alias qwen3-vl-4b --jinja --port 8080
   ```
   What each flag does: `-c 32768` sets the context window (how many tokens the model can see at once); `-np 1` keeps everything in a single request "slot"; `-ngl 99` pushes all model layers onto the GPU (lower this, e.g. `-ngl 20`, if you run out of VRAM, or drop it entirely to run on CPU); `--alias qwen3-vl-4b` is the name you'll refer to this model by later; `--jinja` is **required** for tool calling to work at all; `--port 8080` is the port this server listens on.
- [ ] **In the second window**, start the embedding server:
   ```powershell
   llama-server.exe -m C:\models\Qwen3-Embedding-0.6B-Q8_0.gguf -c 2048 -b 2048 -ub 2048 -np 1 -ngl 99 --embedding --pooling last --port 8081
   ```
   `--embedding` puts the server in embedding mode instead of chat mode; `--pooling last` is required specifically for the Qwen3-Embedding family (check a different embedding model's card before assuming `last` still applies); `-b 2048 -ub 2048` make sure a whole chunk of text fits in one batch, which matters once you start embedding longer passages in Lesson 2.
- [ ] **In a third, regular PowerShell window** (the one you'll keep using for everyday commands), verify both are up:
   ```powershell
   curl http://localhost:8081/health
   ```
   and open `http://localhost:8080` in a web browser — llama.cpp's own built-in chat UI should load.

**Observe**
- What does each server's log print about `n_ctx_slot`? Does it match the `-c` value you passed on that server's command line?
- How much VRAM does each server report using? Run `nvidia-smi` in your third window while both servers are running.

**Discuss**
- Why does the embedding server need a much smaller context window than the chat server?

## Exercise 3 — Talk to the chat server directly

**Goal:** confirm the OpenAI-compatible API works before any C# code exists.

**Do**

- [ ] **In your third PowerShell window**, send a raw chat request:
   ```powershell
   curl http://localhost:8080/v1/chat/completions `
     -H "Content-Type: application/json" `
     -d '{\"model\":\"qwen3-vl-4b\",\"messages\":[{\"role\":\"user\",\"content\":\"Say hello in one short sentence.\"}]}'
   ```
   (The backtick line-continuations are PowerShell syntax; on bash, just put it all on one line and drop the backticks, or use `curl -s ... | python -m json.tool` to pretty-print the JSON.)
- [ ] **Send a raw embedding request:**
   ```powershell
   curl http://localhost:8081/v1/embeddings `
     -H "Content-Type: application/json" `
     -d '{\"model\":\"qwen3-embedding-0.6b\",\"input\":\"hello world\"}'
   ```

**Observe**
- How long is the returned embedding vector (count the numbers in the `"embedding"` array)? Write this number down — it's the embedding dimension, and you'll need it to sanity-check things in Lesson 2.

## Exercise 4 — Scaffold the .NET project

**Goal:** have an empty but runnable .NET console app, in its own folder, ready for Lesson 1.

**Do**

- [ ] **Create a folder for the project** and create the project inside it, e.g.:
   ```powershell
   cd C:\Repos
   dotnet new console -n LocalAgent
   cd LocalAgent
   ```
   Everything from here on runs from inside this `LocalAgent` folder (the one containing `LocalAgent.csproj`) unless a step says otherwise.
- [ ] **Add the packages** that give you `IChatClient` / `IEmbeddingGenerator` over an OpenAI-compatible HTTP API:
   ```powershell
   dotnet add package Microsoft.Extensions.AI
   dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
   ```
- [ ] **Open `Program.cs`** (created by `dotnet new console`, in the project root) and replace its contents with something like:
   ```csharp
   using Microsoft.Extensions.AI;
   using OpenAI;
   using System.ClientModel;

   var client = new OpenAIClient(
           new ApiKeyCredential("not-needed-for-local-llama-server"), // llama-server doesn't check this; OpenAIClient just requires some value
           new OpenAIClientOptions { Endpoint = new Uri("http://localhost:8080/v1") }) // points at your local server instead of OpenAI's cloud
       .GetChatClient("qwen3-vl-4b") // must match the --alias you passed to llama-server
       .AsIChatClient(); // adapts the OpenAI-specific client to Microsoft.Extensions.AI's generic IChatClient

   var response = await client.GetResponseAsync("Hello");
   Console.WriteLine(response.Text);
   ```
   This is scaffolding only — no tools, no RAG, just proof that your C# process can reach the chat server.
- [ ] **Run it** from the project root:
   ```powershell
   dotnet run
   ```

**Observe**
- Does the printed reply match what you got from the raw `curl` request in Exercise 3? It should be a similar kind of answer, even if the exact wording differs (the model isn't deterministic unless `Temperature` is 0).

**Discuss**
- `OpenAIClient` normally talks to OpenAI's cloud API. What single piece of configuration made it talk to your local `llama-server` instead?
