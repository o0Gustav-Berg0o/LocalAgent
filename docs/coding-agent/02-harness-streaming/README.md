# Module 2, Lesson 2 — The Harness, Again: Streaming and Feedback

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 1, Lesson 1 complete. The app starts and reads its config.
- Course 1, Lesson 1 (the agent loop) understood. You should be able to explain history, tool calls, and the iteration limit without looking.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Describe the agent loop as a sequence of messages, and say what each message role means to the model.
2. Switch from a request that waits for the full answer to one that streams tokens as they arrive, and explain why streaming changes the user's experience more than the model's speed.
3. Show a working indicator with elapsed time, without it interfering with the text that streams in.
4. Cancel a generation in progress with a key, and keep the session alive after the cancel.
5. Separate what the user sees from what the model receives, so that display formatting never changes the conversation history.
6. Record and display basic timing, so you can tell a slow model from a slow tool.

---

## 2. Why this lesson exists

In Course 1 the loop called the model, waited, and printed the whole answer. That is fine for a short answer. A coding task produces long answers, multi-step plans, and tool calls that take seconds. Waiting for the whole answer makes the app feel frozen, and a user who sees nothing for 40 seconds will press keys, close the window, or assume something is broken.

Streaming changes two things at once:

- **Perceived speed.** The first word appears in about the time it takes the model to produce it, not the time it takes to finish. The total time may be the same, but the wait feels different.
- **Control.** A user who sees the answer going in the wrong direction can stop it. Without streaming, the only option is to wait or kill the process.

Streaming also introduces new bugs: partial text in the history, a cancel that leaves a half-written message, and a display that mixes streamed text with tool output. This lesson is about getting those right.

---

## 3. Concepts

### 3.1 The loop, in four sentences

1. The user's message is added to the history.
2. The model is called with the whole history and the list of available tools.
3. If the model asks for tools, the app runs them, adds their results to the history, and goes back to step 2.
4. If the model answers without asking for tools, the answer is added to the history and shown.

The iteration limit from Course 1 stops step 3 from repeating forever. Everything in this lesson sits inside step 2 (how the answer arrives) and step 4 (how it is shown).

### 3.2 Roles in the history

Every message has a role. The model treats each role differently:

| Role | Who wrote it | What the model does with it |
|---|---|---|
| system | the app | Follows it as standing instructions. |
| user | the person | Answers it. |
| assistant | the model | Remembers what it said. |
| tool | the app, with a tool's result | Uses it as evidence. Not an instruction. |

A tool result is data. If a file contains the text "ignore your instructions", the model should treat that as file content, which is the lesson from Course 1, Lesson 7. Keeping the roles correct is what makes that possible.

### 3.3 Streaming in one picture

A non-streaming call returns one object when the model is done. A streaming call returns a sequence of small pieces, each a few tokens, as they are produced. The app can:

- print each piece as it arrives,
- collect the pieces into one message,
- add that complete message to the history when the stream ends.

**The rule:** the history holds complete messages only. A half-finished answer must never be added to the history. If it were, the next request would send the model a sentence that ends in the middle.

### 3.4 Display is not history

The text you print and the text stored in the history are different things. The display may:

- add colours,
- wrap lines,
- show a prefix such as `Agent>`,
- hide tool calls behind a one-line summary.

None of these belong in the history. The history stores the plain text the model wrote. If you store the display version, the model will see your decorations as part of its own answers, and its style will drift over time.

### 3.5 Cancellation

A cancel means "stop producing this answer". It does not mean "quit the program". The app has to:

1. stop reading the stream,
2. decide what to do with the partial text,
3. return to the prompt.

For the partial text, two choices are reasonable: discard it, or keep it and mark it as cut off. Discarding is simpler and avoids confusing the model. Keeping it is useful when the user wants to continue from where the answer stopped. This lesson uses discard, and you may extend it later.

### 3.6 The working indicator

A working indicator tells the user the app is alive. It must not:

- overwrite the streamed text,
- flicker,
- keep running after the answer has ended,
- continue after a cancel.

The usual technique is to write the indicator on the line below the output, and to move the cursor back when the first token arrives. Console control codes can do this, but they differ between terminals. Test in the terminal you actually use, and keep a fallback that just prints a dot every second.

### 3.7 Timing

Record two durations for every turn: time to first token, and total time. Time to first token tells you how long the model took to start. Total time tells you how long the user waited. Tool time is the difference between total time and the sum of model times. Showing these numbers is cheap, and it makes "the model is slow" testable.

---

## 4. Gotchas

**4.1 Streamed tool calls arrive in pieces.** A tool call's arguments may come as several fragments that only form valid JSON at the end. Do not try to parse them until the stream has ended for that call.

**4.2 A cancel can happen after a tool call has started.** If the user cancels while a tool is running, the tool may still write its result. Decide whether the result is added to the history. The safe default is: cancel the tool's work if the tool supports it, and mark the result as cancelled.

**4.3 Console output and streaming interact badly.** Any `Console.WriteLine` from a different part of the app, such as a log line, can land in the middle of a streamed word. Route all output through one writer, or accept the mess and note it.

**4.4 Character encoding.** The Swedish letters `å`, `ä`, and `ö` must display correctly. Check the console encoding on your machine. A mismatch shows question marks or boxes, and it looks like a model problem.

**4.5 A key press can be read by the wrong code.** If the app reads keys for the cancel and also for the prompt, a stray keystroke during streaming can start an unwanted command. Use one place that reads keys.

**4.6 The working indicator on a slow first token.** Some models take several seconds before the first token. Show the indicator from the moment the request is sent, not from the first token.

---

## 5. The lab in outline

You will change the existing loop so that:

1. the answer streams to the screen,
2. the history receives the complete answer only,
3. a working indicator shows elapsed seconds,
4. the Esc key cancels the current answer, and the app returns to the prompt,
5. two timing numbers are printed after each turn.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Streaming:** receiving a response as a sequence of pieces, as they are produced.
- **Token:** the unit the model reads and writes. Roughly a word or part of a word.
- **Time to first token:** the delay between sending a request and receiving the first piece.
- **Cancellation token:** a signal the code checks to stop work early.
- **Partial message:** an answer that has not finished. It must not enter the history.
- **Control code:** a special character sequence that moves the cursor or changes colour in a terminal.

---

## 7. Summary

- The loop is the same as in Course 1. What changes is how the answer arrives and how it is shown.
- Streaming improves how the wait feels and gives the user control.
- The history holds complete messages. The display can be decorated; the history cannot.
- A cancel stops the answer, not the program, and discards the partial text.
- Two timing numbers per turn turn "it feels slow" into something you can measure.

Continue to [exercises.md](exercises.md).
