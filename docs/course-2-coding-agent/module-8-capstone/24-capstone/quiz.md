# Module 7, Lesson 24 — Quiz

Write your own answer to each question first, then check the answer key.

1. Why must each task have a check that a script can run?
2. Why must the sample project be reset between tasks, not only between runs?
3. Why should you write the expected effect of a change before you run it?
4. A change improves the success rate and doubles the tokens. Is it a good change? What would you need to know?
5. Why are the safety events counted separately from the success rate?
6. What does a small evaluation set prove, and what does it not prove?

---

## Answer key

1. A check removes the human judgement from the result, so that the same run always gives the same outcome. Without it, the measurement depends on how the answer is read.
2. A task that inherits a change from the task before it measures the combination of the two, not the task. Each task must start from the same baseline.
3. Writing the expectation first protects against reading the numbers in the light of a story you already believe. The comparison between expectation and result is itself a finding.
4. It may be good or bad. Success is only one number. The cost in tokens and time matters, and a change that makes the agent expensive or slow may not be worth it. Decide what trade-off is acceptable before you judge.
5. A high success rate can hide a safe-looking run that was in fact unsafe, or a run where a prompt should have appeared and did not. Safety events are a separate measure, and a regression in them is a regression however good the success rate.
6. It shows the agent can do the tasks in the set, under the conditions of the run. It does not prove the agent works on other tasks, other projects, other models, or for users who behave differently.

---

## Scoring

- 5 or 6 correct: the course is complete for you.
- 3 or 4 correct: re-read sections 3.2, 3.4, and 3.7 of the README, then repeat Exercises 1 and 6.
- Fewer than 3: repeat Exercises 2 to 6.
