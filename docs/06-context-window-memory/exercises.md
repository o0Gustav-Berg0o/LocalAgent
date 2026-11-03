# Lesson 6 — Exercises

You'll be restarting the chat-model `llama-server` window (not the embedding one) several times in this lesson — use the same command from Lesson 0 Exercise 2, just changing `-c`. Stop it with `Ctrl+C` in its window, then retype the command with the new value and press Enter.

## Exercise 1 — Overflow the window on purpose

**Goal:** see what actually happens when context runs out.

**Do**

1. **Stop your chat server** (`Ctrl+C` in its window) and restart it with a small context window:
   ```powershell
   llama-server.exe -m C:\models\Qwen3-VL-4B-Instruct-Q4_K_L.gguf -c 2048 -np 1 -ngl 99 --alias qwen3-vl-4b --jinja --port 8080
   ```
2. `dotnet run`, and ask a question that triggers a `search_notes` call returning several chunks (anything that worked in Lesson 4).

**Observe**
- What happens? Check both your console app's output and the chat server's own terminal window — does it print an explicit error, or does the request seem to succeed with a degraded/truncated response?
- **Restart the server with `-c 32768` again before continuing** — you need the full window for the rest of this lesson.

## Exercise 2 — Watch history grow

**Goal:** connect trace output to actual token cost.

**Do**

1. Confirm your chat server is back on `-c 32768` (restart it if you didn't already).
2. **Add a rough token estimate to your existing `[llm]` trace line** in `AgentLoop.cs` (character count ÷ 4 is a crude but workable estimate for English text):
   ```csharp
   var approxTokens = _history.Sum(m => (m.Text?.Length ?? 0)) / 4;
   Console.WriteLine($"[llm] call {i + 1} — {_history.Count} messages, ~{approxTokens} tokens");
   ```
3. `dotnet run` and ask ten different search-requiring questions in a row (don't `/reset` between them).

**Observe**
- How does the message count and estimated token count grow per turn, from the printed trace lines? Does it grow roughly linearly, or does something jump?

## Exercise 3 — Trimming strategies, compared

**Goal:** compare blunt trimming against a smarter alternative.

**Do**

1. **Add message trimming to `AgentLoop.AskAsync`**, right after you add the user's message to `_history`:
   ```csharp
   private const int MaxHistoryMessages = 6; // start low on purpose for this test
   // ...
   while (_history.Count > MaxHistoryMessages)
       _history.RemoveAt(0);
   ```
   Set `MaxHistoryMessages` to `6`, `dotnet run`, and ask a question, then a few unrelated filler questions, then a follow-up that references your first answer by name (e.g. "what did you say about X earlier?"). Does it still remember?
2. **Set `MaxHistoryMessages` to `80`** and ask ten search-heavy questions in a row. Before you start, estimate (from your Exercise 2 numbers) roughly which question number will push you past 32,768 tokens — then run it and see how close your prediction was.
3. **Implement the improved strategy.** Instead of removing old messages outright, walk the history from oldest to newest and replace the *content* of old tool-result messages with a short placeholder, once history exceeds some threshold, while keeping the question/answer pairs themselves intact:
   ```csharp
   if (_history.Count > MaxHistoryMessages)
   {
       foreach (var msg in _history.Take(_history.Count - MaxHistoryMessages))
       {
           if (msg.Role == ChatRole.Tool)
               msg.Contents = [new TextContent("[old result removed]")];
       }
   }
   ```
4. Repeat the follow-up-question test from step 1 with this new strategy in place.

**Observe**
- Does the placeholder approach let the model keep the thread of conversation better than hard-dropping whole turns, at a similar or lower token cost?

**Discuss**
- When would summarizing old history (instead of trimming it) be worth the extra complexity and cost?
- At what point does "more context" stop helping, even if you have room for it? (Hint: think about small models and prompt length, not just token limits.)
