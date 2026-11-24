# Module 4, Lesson 10 — The Context Budget

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 2 to 5: the loop, the provider, the settings, and the read tools.
- Module 3, Lesson 9: the shell tool, whose output is the largest single source of context growth.
- the RAG course, Lesson 6: the context window as a finite resource.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. List every part of a request that uses the context window, and estimate the size of each.
2. Compute a budget: the context size minus a reserve for the answer, and explain why the reserve must exist.
3. Estimate tokens for text when no tokenizer is available, and state the error of the estimate.
4. Calibrate the estimate against the token counts the server reports, when it reports them.
5. Show the budget to the user as a percentage with a clear colour-independent form, and set thresholds for warnings.
6. Decide what the app does at each threshold, and write the decision as a rule.

---

## 2. Why this lesson exists

Every request to the model sends everything the model needs to answer: the system prompt, the tool descriptions, the whole conversation so far, and the results of every tool call. That total must fit in the context window. When it does not, the server either refuses the request or drops the beginning of the input silently, and in both cases the model loses information without telling anyone.

A coding session grows fast. Each file read adds lines, each command adds its output, and each answer adds more. A user who works for an hour can reach the limit without noticing, and the first sign of trouble is an answer that ignores something said twenty minutes earlier. A budget makes the growth visible before it causes that failure.

---

## 3. Concepts

### 3.1 What uses the window

A request contains five groups of text. Each one has a cost:

| Part | What it is | How it grows |
|---|---|---|
| system prompt | the standing instructions | fixed, but every line costs on every request |
| tool descriptions | the name, description, and schema of each tool | fixed per set of tools; grows with each tool added |
| memory (later) | project notes and skills loaded this session | grows with what is loaded |
| history | every user and assistant message so far | grows with every turn |
| tool results | the output of each tool call, kept in the history | often the largest part, grows with every read and command |

The budget must count all five. A student who counts only the history is surprised by a request that is much larger than expected, and the surprise is always the tool results.

### 3.2 The budget

The budget is the context size minus a reserve:

```text
available for input  =  context size  -  reserve for the answer
```

The reserve is the room the model needs to write its answer. If the input fills the whole window, the model has no room to answer, and the server stops it early or refuses the request. A reserve of 1024 to 2048 tokens is a reasonable starting point for the course, and it is a setting, not a constant.

### 3.3 Estimating tokens

A tokenizer splits text into tokens, and each model has its own. Without the model's tokenizer, a common estimate is about four characters per token for English text. The estimate is rough:

- code tends to produce more tokens per character than prose,
- Swedish text with many special letters can produce more tokens per character,
- numbers, symbols, and long identifiers are expensive.

The honest statement for students: "The estimate can be off by a large margin. Use it to warn early, and use the server's count, if available, to correct it."

### 3.4 Calibration

Some servers report the number of input tokens for each request, in the response. If yours does, the app can compare its estimate with the count, and keep a running correction factor. A correction factor of 1.2 means the estimate is 20 percent low for this model and this kind of text.

Calibration is only as good as its data. A factor based on one short answer is noise. Use a rolling average over several requests, and do not let one unusual request change the factor much.

### 3.5 Showing the budget

The user needs to see the budget without reading a log. A short line in the status area works:

```text
context 42%  (3400 of 8192 tokens, reserve 1024)
```

A percentage alone is not enough, because the user may not know the window size. Show the numbers. Colour can add emphasis, but the percentage and the word must carry the meaning, because some users read with `NO_COLOR` (Module 2, Lesson 4).

### 3.6 Thresholds

A threshold is a point at which the app does something different. For this lesson:

| Level | Share of budget used | What the app does |
|---|---|---|
| normal | below 70% | nothing |
| warn | 70% to 85% | shows the budget in the status line, in a clearer form |
| act | 85% and above | tells the user, and offers compaction (Lesson 11) |
| limit | a request that would exceed the budget | refuses to send it, and asks the user to compact or clear |

The thresholds are settings, and their numbers are a design decision. Write the reason for each one in the README of your app.

### 3.7 What the budget does not do

The budget does not make the model remember more. It only tells you when the window is full. Making the history fit is Lessons 11 and 13's job. Students sometimes expect a budget to fix a problem; it reveals the problem early enough for a decision.

---

## 4. Gotchas

**4.1 The budget counts the answer that has not been written yet.** A reserve that is too small lets the input fill the window, and the answer is cut off mid-sentence. The reserve is part of the budget, not an afterthought.

**4.2 Tool descriptions are counted every time.** Fifteen tools with long descriptions can take thousands of tokens before the user types anything. Measure the cost of the tool list once, and show it as its own line.

**4.3 A calibration factor that drifts.** A factor learned from one kind of text (code) applied to another (Swedish prose) gives wrong numbers. Keep factors per kind of content, or accept the error and say so.

**4.4 The budget differs between models.** Changing the model changes the window and the tokenizer. The budget must be computed from the model in use, not from a number written in the code.

**4.5 Counting characters for bytes.** A file of 32 KB may be 32,000 characters or fewer, depending on encoding. Count what is sent, not what is stored.

**4.6 Thresholds that trigger on every turn.** A warning shown at 70% that repeats every turn trains the user to ignore it. Show a warning once when the level changes, and keep the status line as the continuous display.

---

## 5. The lab in outline

You will:

1. list the five parts of a request and measure each one,
2. compute the budget with a reserve, and show it in the status line,
3. calibrate the estimate against the server's counts, if available,
4. set thresholds and write the action for each,
5. check the budget before sending, and refuse a request that would exceed it,
6. test the boundary cases.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Context window:** the maximum number of tokens a model accepts in one request.
- **Reserve:** the tokens kept free for the model's answer.
- **Budget:** the context window minus the reserve.
- **Tokenizer:** the part of a model that splits text into tokens.
- **Calibration:** adjusting an estimate using measured counts.
- **Threshold:** a level at which the app changes what it does.

---

## 7. Summary

- Five parts use the window: system prompt, tool descriptions, memory, history, and tool results.
- The budget is the window minus a reserve for the answer.
- Estimates are rough; calibration uses the server's counts when they are available.
- The budget is shown with numbers and a word, not colour alone.
- Thresholds decide what happens; each one is a written rule.

Continue to [exercises.md](exercises.md).
