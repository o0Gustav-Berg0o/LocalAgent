# Module 5, Lesson 14 — Quiz

Write your own answer to each question first, then check the answer key.

1. What does a mode change, and what does it not change?
2. Why does cowork mode write the plan before running any step?
3. Why is the plan read at each gate instead of once at the start?
4. A step is marked done in the plan. The agent wants to change its text. What should happen?
5. A user says code mode is "the unsafe mode". What is wrong with that statement?
6. A gate has no answer for twenty minutes. What should the app do, and why?

---

## Answer key

1. A mode changes the gate: when the loop waits for a person and how it paces the work. It does not change the model, the settings, or the permission rules.
2. So the user can see and agree to the whole task before any action happens. A plan the user has not approved must never run a step.
3. The user may edit the plan while the agent works. Reading it at each gate means the latest version is the one that runs.
4. The agent must not change it. A done step records what happened, and changing it would falsify the record. The user may change it, and the app keeps the user's text.
5. Code mode runs the same permission rules as cowork mode. It differs in the plan and the step gates, not in the checks.
6. It should wait. An agent that proceeds without an answer has turned the gate into a formality, and the user's decision was never made.

---

## Scoring

- 5 or 6 correct: ready for Lesson 15.
- 3 or 4 correct: re-read sections 3.1, 3.3, and 3.4 of the README, then repeat Exercises 3 and 4.
- Fewer than 3: repeat Exercises 1 to 4.
