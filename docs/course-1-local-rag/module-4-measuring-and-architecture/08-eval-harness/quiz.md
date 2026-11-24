# Lesson 8 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. Why isn't "it feels better" a valid way to compare two retrieval configurations?
2. What does recall@k measure?
3. Why does recall@k deliberately not check whether the model's final answer was correct?
4. Why should some of the eval set's questions have an empty "expected sources" list?
5. What makes an evaluation "rigorous" compared to casually trying a few queries?

## Answers

1. Because you likely tested different settings on different days with different queries and no fixed ground truth, so you can't attribute any difference to the one thing you actually changed.
2. Among the top-k search results for a question, whether at least one of that question's known-correct expected sources appeared — averaged over a fixed set of test questions into one comparable number.
3. To isolate retrieval quality from everything the chat model might do with the retrieved material afterward — it deliberately tests one layer (search) in isolation.
4. To test that your system correctly handles the case where nothing in your notes actually answers a question — that absence is detected properly, not just that presence is found correctly.
5. Fixing the questions and their correct answers first, then running every configuration against that exact same fixed test, so any score difference is attributable to the one thing that changed, not to different queries or different days.
