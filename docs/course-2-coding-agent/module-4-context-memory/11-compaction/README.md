# Module 4, Lesson 11 — Compaction

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 4, Lesson 10: the budget, the thresholds, and the refusal before sending.
- Module 2, Lesson 2: the history holds complete messages only.
- the RAG course, Lesson 6: basic trimming and placeholder replacement, which compaction replaces.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain the difference between trimming, placeholdering, and compaction, and when each is the right tool.
2. Design a summary format with fixed sections, so that the important facts survive.
3. Decide what stays verbatim: pinned instructions, the most recent turns, and facts a later turn will need.
4. Run compaction as a model call, and account for its cost and its failure modes.
5. Keep the full transcript on disk, so a compaction never destroys the only copy of the conversation.
6. Test a compaction against a list of facts that must survive, and measure how many did.

---

## 2. Why this lesson exists

Lesson 10 shows the user when the window is full. Something has to happen next. There are three options, and they are not equal:

- **Trimming** removes the oldest messages. It is cheap and simple, and it removes whatever was said first, even when it was the most important thing. A rule set at the start of a session disappears first.
- **Placeholdering** replaces old tool results with a short note (the RAG course, Lesson 6). It frees the largest part of the window, but it keeps the conversation's shape, and the model still knows that a file was read, just not what it said.
- **Compaction** asks the model to summarise the older part of the conversation into a short, structured note, and keeps the recent part exactly as it was. It keeps the meaning of older turns at the cost of one model call and some accuracy.

A good agent uses all three, in order of cost: placeholder first, then compact, and trim only as a last resort. This lesson builds the compaction step and the rules that decide between them.

---

## 3. Concepts

### 3.1 What a summary must keep

A summary is only useful if the next turn can act on it. A summary that says "we discussed the project structure" is worthless. A useful summary keeps:

- **the goal:** what the user is trying to achieve, in one sentence,
- **the decisions:** what was chosen, and why, when the reason matters,
- **the files:** which files were read or changed, and what changed in each,
- **the open items:** what is not done yet,
- **the constraints:** rules the user set ("do not touch the tests", "use snake_case") that must still hold.

A fixed section layout makes sure each of these is considered every time. A free-form summary tends to drop the constraints first.

### 3.2 What stays verbatim

Some messages are never summarised:

- **Pinned instructions:** the project instruction file and any rule the user marked as permanent (Lesson 12). These are not in the history at all, but if they were, they would stay verbatim.
- **The most recent turns:** the last few exchanges, so the model keeps the thread of the current question. A common default is the last four turns.
- **The current task's tool results,** if the current answer still uses them.

Everything older than that is eligible for compaction.

### 3.3 Compaction as a model call

Compaction asks the model to write the summary. That is a request with its own budget: it includes the old messages and the summary instructions, and it produces a summary as output. It has three costs:

1. time: a summary call takes as long as an answer,
2. accuracy: the model may leave out or invent facts,
3. risk: a summary is a new text the model will trust later, so an error in it becomes an error in every later answer.

Because of the third cost, the summary must be shown to the user, or at least stored where the user can read it.

### 3.4 Keep the full transcript

Compaction replaces messages in the working history. It must not delete the original messages. The full transcript is written to the session file (Lesson 13) before compaction, so that:

- the user can read what was said,
- a bad summary can be redone from the original,
- a debugging session can see what the model actually saw.

The working history is a view. The transcript is the record.

### 3.5 When to compact

Compaction is triggered by the budget (Lesson 10). The rule is:

- at the **act** threshold, offer compaction; the user decides,
- when the user runs `/compact`, compact now,
- if the user has enabled automatic compaction, compact at the act threshold without asking, and say so.

Automatic compaction is a setting, and it is off by default. An unexpected summary changes what the model knows, and the user must be able to see that it happened.

### 3.6 Placeholder before compaction

Placeholdering (the RAG course, Lesson 6) frees the most space for the least cost. Apply it first:

1. replace old tool results with a short placeholder,
2. check the budget,
3. only if the budget is still above the act threshold, compact.

This order keeps summaries rare, which keeps their risk low.

### 3.7 Fidelity

A summary's fidelity is how many of the important facts it keeps. You can measure it: write a list of ten facts from a conversation that must survive, compact the conversation, and ask the model a question that needs each fact. Count the correct answers. A fidelity of 8 out of 10 is useful. A fidelity of 4 out of 10 means the summary is not a summary, it is a loss.

---

## 4. Gotchas

**4.1 Summarising the constraints away.** A constraint stated once early in the conversation is the first thing a summary drops. Pin constraints explicitly, or put them in the project memory (Lesson 12), so they are never only in the history.

**4.2 A summary that adds facts.** A model may write a plausible file name or a decision that never happened. Check the summary against the transcript for each file name and number.

**4.3 Compacting in the middle of a tool loop.** If compaction runs between a tool call and its result, the result has nowhere to go. Compact only between complete turns, after all tool results for the last answer are in the history.

**4.4 Compacting twice.** A summary that is itself summarised later loses detail each time. Keep the summary as one message, and when compacting again, summarise the old summary together with the messages after it, and say so in the format.

**4.5 The summary call exceeds the budget.** The compaction request includes the old messages, so it can be larger than the window. Split the old messages into chunks if needed, and summarise each, then merge.

**4.6 Losing the transcript.** A compaction that overwrites the session file destroys the record. Write the transcript first, and check that the write succeeded.

---

## 5. The lab in outline

You will:

1. design a summary format with fixed sections,
2. implement placeholdering before compaction,
3. implement compaction as a model call, with the transcript kept first,
4. decide what stays verbatim and implement it,
5. add `/compact` and the automatic option,
6. measure fidelity with a list of facts.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Trimming:** removing the oldest messages from the history.
- **Placeholdering:** replacing old tool results with a short note.
- **Compaction:** replacing old messages with a summary written by the model.
- **Verbatim:** kept exactly as written.
- **Transcript:** the complete record of the conversation, kept on disk.
- **Fidelity:** how many important facts a summary keeps.

---

## 7. Summary

- Three options: trim, placeholder, compact. Use them in order of cost.
- A summary has fixed sections: goal, decisions, files, open items, constraints.
- Pinned rules and the most recent turns stay verbatim.
- Compaction is a model call, shown to the user, and never destroys the transcript.
- Fidelity is measured against a list of facts that must survive.

Continue to [exercises.md](exercises.md).
