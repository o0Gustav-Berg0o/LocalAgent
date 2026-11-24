# Module 4, Lesson 11 — Exercises: Compaction

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state, with Lesson 10's budget working.
- Set the context size in the config to a small value, such as 4096, so you reach the thresholds within a few questions.
- Start the chat `llama-server`.

---

## Exercise 0 — Reach the threshold on purpose

**Goal:** a long session that hits the act threshold, so you have something to compact.

**Do**

1. Ask five questions about the sandbox project. Each should read a file or search for something, so the history grows.
2. Write down the status line after each question: the percentage, and the used tokens.
3. Note the question at which the act threshold is first crossed.

**Observe**
- Which turn added the most? Which tool result was the largest?

**Checkpoint:** you have five status readings, and you know the turn at which the act message appeared.

---

## Exercise 1 — Placeholdering first

**Goal:** before any summary, free space by replacing old tool results with placeholders. Measure how much this frees.

**Do**

1. Create `Context/Placeholder.cs`. It replaces the content of tool results older than the most recent N turns (default 4) with a placeholder that says what was done: "[result of read_file on src/Program.cs, 300 lines, removed to save space]".
2. Keep the call identifier in each replaced result (the RAG course, Lesson 6 fixed this bug). The placeholder must still be a tool result, not plain text.
3. Make the function report how many characters it freed.
4. Run it on the Exercise 0 history, and check the status line afterwards.

**Read this carefully.** The placeholder must say what the result was, not only that it was removed. A model that sees "[removed]" does not know whether it read a file or ran a command, and it may read the file again, which costs the same tokens.

**Checkpoint:** after placeholdering, the status line shows a lower percentage, and the history still has one tool message for each call.

---

## Exercise 2 — The summary format

**Goal:** a fixed structure the summary must follow, so that no section is forgotten.

**Do**

1. Create `Context/SummaryFormat.cs` with the structure: goal, decisions, files, open items, constraints. Write the section names and one sentence describing each.
2. Write the instructions for the summary model call. They must say: use exactly these sections, keep file names exactly as written, do not add facts that are not in the messages, and say "none" for an empty section.
3. Keep the instructions short. They are sent with every compaction, and they cost tokens.

Here is the shape of the instruction text. Adapt the wording, but keep the rules:

```text
Summarise the conversation below for continuing the same task.
Use exactly these headings: Goal, Decisions, Files, Open items, Constraints.
Under Files, list every file name exactly as it appeared. Do not add any file name that did not appear.
Under Constraints, keep every rule the user stated, word for word where possible.
If a heading has nothing to report, write "none".
Do not add facts that are not in the messages.
```

**Checkpoint:** the instruction text is under 150 words, and each rule in it corresponds to a failure you have seen or can imagine.

---

## Exercise 3 — Compaction as a model call

**Goal:** the old messages are sent to the model with the summary instructions, and the reply replaces them in the working history.

**Do**

1. Create `Context/Compactor.cs`. It takes the messages to compact and returns a summary message.
2. Before calling the model, write the full transcript to the session file (you may use a simple text file until Lesson 13). Check that the write succeeded. If it did not, stop, and do not compact.
3. Call the model with the summary instructions and the old messages. Use the non-streaming call from Module 2, since the summary is not shown token by token.
4. Replace the old messages in the working history with one message, marked as a summary. The recent turns stay as they were.

**Read this carefully.** The order is: write the transcript, then call the model, then replace the history. If the model call fails, the history must be unchanged. A compaction that fails halfway must leave the conversation as it was.

**Checkpoint**
- After compaction, the history has one summary message followed by the recent turns.
- The transcript file contains every original message.
- A simulated failure of the model call leaves the history unchanged.

---

## Exercise 4 — What stays verbatim

**Goal:** the recent turns, the pinned rules, and the current tool results are not summarised.

**Do**

1. Make `Compactor` split the history into three parts: old messages (to summarise), recent messages (kept verbatim, default the last four turns), and nothing else. A turn is a user message and everything the model did in answer to it.
2. Make the split respect turn boundaries. Never cut a turn in half, and never separate a tool call from its result (Exercise 3 of Module 2, Lesson 2 covered the same rule).
3. Add a pinned-rules list to the config, for rules the user wants to survive every compaction. Those rules are added to the summary instructions as a fixed block, not summarised.

**Read this carefully.** A split in the middle of a turn is a common bug. The model then receives a tool result without its call, or a call without its result, and it may refuse the next request.

**Checkpoint**
- Compacting a history of ten turns leaves the last four turns exactly as they were.
- A pinned rule appears in the summary, word for word, under the constraints heading.

---

## Exercise 5 — `/compact` and the automatic option

**Goal:** the user can compact on demand, and can choose to let the app compact automatically.

**Do**

1. Add `/compact` to `CommandHelper`. It runs placeholdering, then checks the budget, and compacts only if still above the act threshold. It prints what it did and how much space was freed.
2. Add an `autoCompact` setting, default off. When it is on and the act threshold is crossed, compact automatically, and print one line saying so.
3. Make sure `/compact` with nothing to compact prints "nothing to compact" and changes nothing.

**Checkpoint**
- `/compact` on a short history prints "nothing to compact".
- `/compact` on a long history prints what was placeholdered, what was summarised, and the new percentage.
- With `autoCompact` on, crossing the act threshold compacts and prints the line; with it off, the user is only offered `/compact`.

---

## Exercise 6 — Measure fidelity

**Goal:** a number for how much the summary keeps, so you can judge compaction with evidence.

**Do**

1. Before the conversation starts, write a list of ten facts that will be established during it. Include file names, a decision, a number, and one rule. Example facts: "the test file is named Parser.Tests.cs", "the user wants no new packages", "the default port is 8081".
2. Run the conversation to establish the facts. Compact.
3. Ask ten questions, one per fact, each answerable only from the fact. Record the answer for each as right or wrong.
4. Write the fidelity: the number right, out of ten.

**Read this carefully.** The questions must be answerable only from the fact. If a question can be answered from the model's general knowledge, the score measures luck.

**Checkpoint:** you have a fidelity score, and a list of the facts that were lost.

**Observe**
- Which kind of fact was lost most often: file names, numbers, decisions, or constraints? Why would that kind be lost?

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Compact twice | compact, have more turns, compact again | the second summary includes the first, labelled |
| Compact during a tool loop | trigger compaction between a call and its result | not possible; compaction waits for the turn to end |
| Model call fails during compaction | stop the server during the summary | history unchanged, error shown |
| Transcript cannot be written | make the session folder read-only | compaction refused, history unchanged |
| Summary invents a file name | a summary with a file name not in the conversation | detected by the check, or documented as a known limit |
| Pinned rule missing from summary | remove the pinned rule from the config and check | the rule is absent; the warning explains it must be pinned |
| Very long history | a history larger than the window | old messages summarised in chunks, then merged |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Compare the status line before placeholdering, after placeholdering, and after compaction. Which step freed the most, and which cost the most?
2. Would you trust the summary without reading it? What would make you check it?
3. A user says the model "forgot" something from earlier. What would you check first: the transcript, the summary, or the recent turns?

---

## Discuss

- A summary is a new text that the model will trust. Who should be responsible for checking it, and how would the user know to check?
- Placeholdering keeps the shape of a conversation and removes its content. When does the shape matter more than the content?
- The transcript on disk makes compaction reversible. What does that change about how a user should feel about it?

---

## Stretch

- Add a "show summary" command that prints the current summary, and a "redo summary" command that recreates it from the transcript.
- Compare fidelity with two different summary instructions. Which change helped, and by how much?
- Summarise in chunks in parallel, when the history is larger than the window. What must be true for the chunks to be merged safely?

---

## Time check

If you are past the planned time, finish Exercises 1, 3, and 4. Placeholdering, the transcript-first rule, and the verbatim split are what keep compaction from losing the user's work.
