# Module 4, Lesson 10 — Exercises: The Context Budget

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state, with Module 3 working.
- Start the chat `llama-server`. Note the context size you started it with (Lesson 3's `serverContextSize`).
- Find out whether your server reports input token counts in its responses. Send one request and inspect the response fields. Write down what you find, even if it is "no count reported".

---

## Exercise 0 — Find out the real size of a request

**Goal:** a number for the actual size of what is sent, before you estimate anything.

**Do**

1. Ask a short question with no tools, and note the response's token count if it reports one. If it does not, skip to step 3.
2. Add a temporary print that writes the total characters of every message in the request, and the number of messages.
3. Ask the same question with three tools registered, then with nothing registered. Note the characters in each case.
4. Ask a question that makes the model read a file of about 300 lines. Note the characters again, and the increase.

**Observe**
- Which step increased the request size the most? Was it the history, the tool results, or the descriptions?
- Did the size of the tool descriptions surprise you? How many characters do the descriptions alone add?

**Checkpoint:** you have a table with at least four rows (no tools, three tools, short answer, after a file read), with characters for each.

---

## Exercise 1 — Measure the five parts

**Goal:** the budget counts each part separately, so a student can see which part is growing.

**Do**

1. Create `Context/ContextMeter.cs`. It computes the characters of each part of a request: the system prompt, the tool descriptions, the memory (empty for now), the history, and the tool results inside the history.
2. Separate tool results from other history, by the message role (Module 2, Lesson 2). A tool message counts as a tool result.
3. Return the five numbers as a record. Do not return a single total; the parts are the point.

**Read this carefully.** The tool results are inside the history. If you count them twice (once in history, once as a separate part), the budget is wrong by the size of every file read. Count each message once, and assign it to one part.

**Checkpoint:** after the Exercise 0 question with a file read, the meter shows a larger tool-result number than before, and the history number grows by the same amount in the next turn.

---

## Exercise 2 — Estimate tokens

**Goal:** a token estimate for each part, with the estimate's rule written in code.

**Do**

1. Create `Context/TokenEstimator.cs` with a method that converts characters to an estimated token count. Start with four characters per token, and keep the divisor in one named constant.
2. Write in a comment what the estimate is good for and what it is not. Include the Swedish and code caveats from the README (section 3.3).
3. Add a method that returns the estimate for the whole request, as the sum of the five parts.

**Checkpoint:** the estimate for the Exercise 0 request is within the same order of magnitude as the server's reported count, if your server reports one. If the difference is more than a factor of two, write the difference down. Do not change the constant yet.

---

## Exercise 3 — Calibrate against the server

**Goal:** if the server reports token counts, the app learns a correction factor for the model in use.

**Do**

1. If your server reports input tokens, read the count from each response. If it does not, implement the calibrator anyway and test it with recorded numbers (write the numbers in a file for the test).
2. Create `Context/Calibrator.cs`. For each response with a count, compute the ratio of the reported count to the estimate for that request.
3. Keep a rolling average of the last ten ratios. Use the average as the correction factor. Store the factor per model name in the config or a small cache file.
4. Ignore a single ratio that is far from the others (more than three times the average). Write the rule in a comment.

**Read this carefully.** A calibrator learned on code must not be used for Swedish prose without saying so. Store the factor per model, and mention the content kind in the README if you see a difference between kinds of text.

**Checkpoint:** after ten requests, the calibrator reports a factor, and the estimate multiplied by the factor is closer to the reported count than the raw estimate. If your server reports no counts, the test with recorded numbers passes.

---

## Exercise 4 — The budget and the status line

**Goal:** the status line shows the used share of the budget, with the numbers beside the percentage.

**Do**

1. Compute the budget: the context size from the config, minus the reserve from the config (default 1024).
2. Compute the used share: the corrected estimate divided by the budget, as a percentage, rounded down.
3. Show it in the status line: the percentage, the used tokens, the budget, and the reserve. Example: `context 42%  (3400 of 7168 tokens, reserve 1024)`.
4. Make the line update after each answer. Do not update it during streaming; a number that changes every token is noise.

**Read this carefully.** The budget is the window minus the reserve, so the denominator is 7168 in the example, not 8192. Students often divide by the window, which hides the reserve.

**Checkpoint**
- The status line shows a percentage that rises after a file read.
- The numbers in the line are consistent: used divided by budget gives the percentage.
- With `NO_COLOR` set, the line is identical in content.

---

## Exercise 5 — Thresholds and their actions

**Goal:** each threshold has a written action, and the action happens once when the level changes.

**Do**

1. Add the thresholds to the config: warn at 70, act at 85, and the reserve. Validate them (the warn value must be lower than the act value).
2. When the used share crosses the warn threshold upward, print a single line: "Context is 70% full. Long answers and file reads will fill it faster."
3. When it crosses the act threshold, print: "Context is 85% full. Use /compact (coming in Lesson 11) or /clear to free room." At this lesson, the message may name the command without implementing it.
4. Do not repeat the message while the level stays the same. Repeat it only after the level drops and rises again.

**Read this carefully.** A threshold that prints every turn is ignored after the second turn. The rule is to print on change, not on state. Test it by staying above the threshold for three questions.

**Checkpoint**
- Crossing 70% prints the warn message once.
- Staying above 70% for two more questions prints nothing new.
- Crossing 85% prints the act message once.

---

## Exercise 6 — Refuse a request that would exceed the budget

**Goal:** the app never sends a request the server cannot accept. If the next request would exceed the budget, the app stops and tells the user.

**Do**

1. Before each request, compute the estimated size of the request including the new user message and the expected tool results (use the estimate, not zero).
2. If the estimate exceeds the budget, do not send. Print: "This request would exceed the context budget. Use /compact or /clear, then try again." This message is a limit, not a warning.
3. Make sure a refused request is not added to the history. The user's message stays in the input line for editing, if your console supports that, and otherwise is printed back so the user can copy it.

**Read this carefully.** The check runs before the request, using the estimate. A request that is slightly under the estimate limit may still fail on the server, if the estimate is low. This is why calibration (Exercise 3) matters.

**Checkpoint:** set the context size in the config to a small value (for example 2048), ask a question that reads a large file, and confirm that the app refuses the next request with the limit message, without calling the server.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Reserve larger than the window | set reserve to the context size | refused by validation, default used |
| Warn above act | set warn to 90, act to 80 | refused by validation, warning shown |
| Server reports no counts | use a server that reports nothing | estimate used, no crash |
| Calibrator with one outlier | record eleven ratios, one very large | outlier ignored |
| Model changed in settings | change the model name | the correction factor for the new model is used |
| Swedish text | ask a question in Swedish about a Swedish note | estimate shown, difference from count noted |
| Empty history | start a new session | status line shows only the system prompt and tools |
| Very long single message | paste a 20,000-character message | refused by the limit check, or accepted and the level shown |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which part of the request grew the most in your session? Was it the part you expected?
2. How wrong was the raw estimate for your model and your text? What would you tell a student about trusting it?
3. Is refusing a request better than sending it and letting the server cut it? Who is protected by each choice?

---

## Discuss

- A budget shows the user a number they cannot control directly. What can they control, and what should the budget tell them to do?
- Why is the reserve a setting, not a fixed value? Who should decide how much room the answer gets?
- The status line is the only place the budget is visible. What else should the user be able to ask for, and where?

---

## Stretch

- Show the five parts as a small breakdown with `/context`, so the user can see which part is largest.
- Add a per-tool cost: the average characters returned by each tool. Use it to warn before a tool with a large result is called.
- Compare the estimate with a real tokenizer for your model, if one is available to you. How large is the difference, and is it worth the dependency?

---

## Time check

If you are past the planned time, finish Exercises 1, 4, and 6. The meter, the status line, and the refusal are the parts that protect the user from silent failure.
