# Module 2, Lesson 2 — Exercises: Streaming and Feedback

Work in order. Each exercise ends with a **Checkpoint**. Do not continue until it passes.

**Before you start**
- Start from a clean git state, so you can return to the working version of the loop.
- Start the chat `llama-server` (port 8080). The embedding server is not needed for this lesson.
- Use a terminal you actually work in. Streaming output looks different in different terminals, and a lab that only works in one terminal is not finished.
- Keep a copy of the current `AgentLoop.cs`. Exercise 0 asks you to compare against it.

---

## Exercise 0 — Measure the current loop

**Goal:** know how the app behaves now, before you change it.

**Do**

1. Start the app, ask a question that needs a long answer, for example "Explain how the turbocharger in my notes works, in detail." Time it with a stopwatch from pressing Enter to the last word appearing.
2. Write down what you see during the wait. Is there any sign that the app is working?
3. Open `Agent/AgentLoop.cs`. Find the call that sends the history to the model. Write down its method name. It is a non-streaming call. Confirm that by checking whether the method returns one complete response.
4. Find the place where the answer is added to the history. Write down the line number.

**Observe**
- Imagine you are a user who has never seen the app. After 20 seconds of silence, what would you do?

**Checkpoint:** you have the wait time, a note on what the user sees during the wait, and the two line numbers. Keep them for Exercise 6.

---

## Exercise 1 — Read the roles

**Goal:** be able to say, for each message in the history, who wrote it and what the model should do with it.

**Do**

1. Add a temporary line to `AgentLoop` that prints the role of every message in `_history` before each model call. The message roles are available on each `ChatMessage`.
2. Ask a question that uses one tool, for example a note search.
3. Read the printed roles in order. Write them down in a list. Expect something like: system, user, assistant (tool call), tool (result), assistant (answer).
4. Remove the temporary line when you have the list.

**Observe**
- Which message did the model write? Which did the app write? Is there any message the user wrote that was not their own question?
- Find one tool result in your list. Does it contain any text that looks like an instruction? What should the model do with that text?

**Checkpoint:** you have the role list for one tool-using turn, and you can say which role the tool result has and why that matters.

---

## Exercise 2 — Stream the answer

**Goal:** the answer appears token by token. The history still receives the complete answer at the end.

**Background:** the chat client in `Microsoft.Extensions.AI` has two call styles: one that returns the whole response, and one that returns an asynchronous sequence of updates. The second style is the streaming one. Each update carries a piece of text. Look up the exact method name and signature in the package you are using; do not copy it from this document without checking, because versions change.

**Do**

1. Create a method `StreamAnswerAsync` in `AgentLoop`. It takes the history and the tools, and returns the complete text when the stream ends.
2. Inside it, read the updates one by one. For each update that carries text, print the text straight away, with no line break, and add it to a `StringBuilder`.
3. When the stream ends, return the `StringBuilder`'s text.
4. Replace the non-streaming call in `AskAsync` with your new method, for the step where the model gives its final answer.
5. Keep the tool-call handling as it is for now. Tool calls arrive in updates too. Do not run them while you are still printing text; collect them and handle them after the stream ends. (Exercise 4 covers this more carefully.)

Here is the shape of the loop body. The method names are placeholders; use the ones your code has:

```csharp
var builder = new StringBuilder();
await foreach (var update in client.GetStreamingResponseAsync(history, options))
{
    // Text pieces are printed as they arrive.
    if (!string.IsNullOrEmpty(update.Text))
    {
        Console.Write(update.Text);
        builder.Append(update.Text);
    }
    // Tool-call pieces are collected for later, not run here.
}
Console.WriteLine();
```

**Read this carefully.** `Console.Write` without `WriteLine` keeps the cursor on the same line, so each piece joins the previous one. The `builder` keeps a copy of the whole text, so the history receives one complete message. The `Console.WriteLine()` after the loop ends the line.

**Checkpoint**
- A long question shows text within a second or two of the model starting (time to first token), not at the end.
- The text on screen matches the text stored in the history. Check by printing the last assistant message after the answer.
- A question that needs no tool still produces one assistant message in the history, not two.

**Observe**
- Compare the total time with Exercise 0. Is the total time longer, shorter, or the same? Why would it differ at all?

---

## Exercise 3 — Keep partial text out of the history

**Goal:** a half-written answer is never stored. This prevents the model from continuing a sentence that was never finished.

**Do**

1. Make sure the history is only updated after the stream ends normally. Find the line where the assistant message is added, and check that it runs after the loop in `StreamAnswerAsync`, not inside it.
2. Simulate an interruption. Add a temporary `throw new Exception("test")` after the first ten updates. Run the app and ask a question.
3. Check what the history contains after the exception. Use the same temporary print from Exercise 1.
4. Remove the test exception.

**Observe**
- After the exception, does the history contain a partial assistant message? If it does, your code stores too early. Move the add to after the loop.
- What does the user see after the exception? Is the error message clear enough to act on?

**Checkpoint:** after a simulated failure in the middle of a stream, the history contains no partial assistant message, and the error is printed.

---

## Exercise 4 — Handle tool calls in a stream

**Goal:** tool calls that arrive in pieces are assembled correctly, and the app runs them only after the model has finished asking.

**Background:** a tool call can arrive as several updates. The name often comes first, then the arguments arrive in fragments. Only the complete arguments form valid JSON. Running a tool with half its arguments is a bug that produces confusing errors, so the app must wait for the stream to end before running anything.

**Do**

1. In `StreamAnswerAsync`, keep a list of tool calls. When an update contains a tool call, add it to the list. If the library gives you the complete call in one piece, use that. If it gives fragments, merge them by call identifier.
2. After the stream ends, run the tools in the order they were requested, add each result to the history, and continue the loop as before.
3. Print a line for each tool call before running it, using the format from the RAG course: `[tool] name(arguments) -> result`.

**Read this carefully.** The model may ask for two tools in one answer. Both calls belong to the same assistant message, and each result must reference the call it answers. If the call identifiers do not match, the model cannot connect results to requests.

**Checkpoint**
- A question that needs a note search runs the search once, after the answer has ended streaming.
- The trace line shows the complete argument, not a fragment.
- The history has one assistant message with the tool call, followed by one tool message with the result.

**Observe**
- What would the trace show if you ran the tool while the arguments were still arriving? Try it on purpose with a temporary change, and write down what you see. Then revert.

---

## Exercise 5 — The working indicator

**Goal:** a visible sign that the app is working, from the moment the request is sent until the answer ends.

**Background:** the indicator must not write over the streamed text. The safe approach in this lesson is a separate line that is cleared when the first token arrives. Test it in your terminal. If the terminal does not support clearing a line, fall back to printing a single dot every second, and say so in your notes.

**Do**

1. Before the request is sent, start a background task that writes `working 0s` on the current line and updates the number every second.
2. When the first text update arrives, stop the background task, clear the line, and write the answer on that line.
3. Make sure the background task stops if the stream ends with no text (a tool-only answer), and also if an exception happens.

Here is the structure. Fill in the details yourself:

```csharp
using var cts = new CancellationTokenSource();
var indicator = ShowWorkingIndicator(cts.Token);   // writes "working Ns" every second
try
{
    // start the stream; on the first text update, call cts.Cancel() and clear the line
}
finally
{
    cts.Cancel();   // always stop the indicator, even after an exception
}
```

**Read this carefully.** The `finally` block is the reason this works. Without it, an exception leaves the indicator running, and it writes over the next prompt.

**Checkpoint**
- The indicator appears within a second of pressing Enter.
- It disappears when the first token arrives, and the answer starts on a clean line.
- A tool-only answer (no text) also clears the indicator, when the tool starts.
- After an exception, no indicator keeps running. Check by asking another question.

**Observe**
- Is the indicator still useful when the first token is fast? Would you keep it for a sub-second answer? Why?

---

## Exercise 6 — Cancel with Esc

**Goal:** pressing Esc stops the current answer and returns to the prompt. The program keeps running.

**Background:** the cancellation token from Exercise 5 can also be used for this. The key must be read by one place only (section 4.5 of the README).

**Do**

1. While the stream is running, check for a key press without blocking. The console offers a way to check whether a key is waiting. Use it in the streaming loop, or in a separate task that only watches for Esc.
2. When Esc is pressed, cancel the token. The stream stops, the partial text is discarded, and no assistant message is added.
3. Print a short line, for example `[cancelled]`, and return to the prompt.
4. Make sure the next question works normally. A cancel must not leave the history in a broken state.

**Read this carefully.** Discarding the partial text means the history after a cancel looks as if the cancelled question was never answered for the model. The user's question stays in the history, so the model knows it was asked. Decide whether that is what you want, and write a sentence in your notes explaining why. (Hint: the alternative is to add a short assistant message such as "[cancelled by user]", which is clearer to the model.)

**Checkpoint**
- Esc during a long answer stops within about a second.
- The prompt returns, and the next question is answered normally.
- The history contains the user's question, and no partial answer.
- Ctrl+C still quits the program. (If it does not, decide whether that is correct, and say why.)

**Observe**
- Press Esc at the very start, before the first token, and again at the very end, after the last token. Do both work? What is the difference for the user?

---

## Exercise 7 — Two timing numbers per turn

**Goal:** after each answer, print time to first token and total time. These numbers separate a slow model from a slow tool.

**Do**

1. Record the time when the request is sent, and the time the first text update arrives. The difference is time to first token.
2. Record the time when the answer ends. The difference from the send time is total time, which includes any tool calls and their model calls.
3. After the answer, print one line, for example: `[timing] first token 1.8s, total 14.2s, tools 3.1s`. Tool time is total time minus the model time you can attribute, so for this exercise, measure tool time directly around each tool call and add it up.

**Read this carefully.** Measuring tool time directly is more reliable than subtracting. The model time includes network waiting and generation, and both are hard to split. The direct measurement is simple: start a stopwatch before each tool, stop it after.

**Checkpoint**
- A question with no tool shows a tools time of zero (or leaves the field out).
- A question with one note search shows a tools time that is small compared with total time.
- The numbers are plausible. A total time shorter than first-token time means a bug. Fix it before continuing.

**Observe**
- Ask two questions of similar length, one that needs a tool and one that does not. Compare their timing lines. Which number surprised you most?

---

## Exercise 8 — Display and history are different

**Goal:** prove that decorations on the screen never reach the history.

**Do**

1. Add a visible prefix to the output, for example a coloured `Agent>` before each answer. Use the colour and prefix from the existing app.
2. Ask a question. Then print the last assistant message from the history, using the temporary print from Exercise 1.
3. Compare the two. The screen shows the prefix; the history must not.

**Observe**
- If the prefix were in the history, what would the model do in the next turn? Describe what you would expect to see in its answers after five turns.

**Checkpoint:** the printed history message has no prefix, no colour codes, and no line decorations.

---

## Exercise 9 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Very long answer | a question that asks for a long essay | streams the whole time, no crash, history complete |
| Empty answer | a question the model answers with nothing | no crash, a short message shown |
| Tool-only answer | a question that only triggers a tool | the indicator clears when the tool starts |
| Two tools in one answer | a question that needs two note searches | both run after the stream, both results in history |
| Network error mid-stream | stop the server during a long answer | error message, no partial message in history, app keeps running |
| Esc twice quickly | press Esc twice | no crash, one cancel |
| Swedish characters | ask a question with `å`, `ä`, `ö` in the question and expect them in the answer | characters display correctly |
| Terminal resize during answer | resize the window while streaming | output continues, no crash |

**Checkpoint:** every row has a written result. Fix any row that failed, and note what you changed.

---

## Observe (whole lesson)

1. Which change in this lesson made the app feel most different? Why does that matter more than the model being faster?
2. Name one place where display and history could easily be confused in your code. How did you keep them apart?
3. A user cancels a tool call in the middle. What should happen to the tool, and to its result? Write your decision.

---

## Discuss

- Streaming makes the wait feel shorter without making it shorter. What other design choices make a tool feel responsive?
- Why does the history hold complete messages only? What would a half-finished message do to the model's next answer?
- Esc cancels a model answer. What would a user expect Esc to do in a cowork session, where the agent is waiting for approval?

---

## Stretch

- Show a short summary of token counts per turn, using the counts the model reports if your server returns them, or an estimate if it does not. Label estimates clearly.
- Add a setting to turn streaming off. When would a user want that?
- Measure the difference between two terminals on the same machine. Write down which one you would recommend to a student.

---

## Time check

If you are past the planned time, finish Exercises 2 and 3 first. Streaming and the history rule are the parts the rest of the course depends on. The indicator and timing can be completed next session.
