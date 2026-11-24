# Lesson 5 — Tool Design for Small Models

**Slides:** https://docs.google.com/presentation/d/1pd9uKUXwhIbtT_x1WmG_aG1VROjN53CZmAJS4xItxAc/edit
**Quiz:** [quiz.md](quiz.md) · https://docs.google.com/document/d/1aZow9pmgvqXwWbZ08C7CR3_2xtASDuXwHpuAx9fFr_c/edit

## Learning objectives

- Understand that every tool definition (name, description, parameter schema) is part of the prompt, not just metadata.
- See directly how small (3–4B) models degrade with bad tool design, in ways larger/cloud models tolerate.
- Learn concrete design rules for tools meant to be used by small, cheap, local models.

## Tool definitions are prompts

Every tool you register gets serialized (name, description, JSON schema) and sent to the model with every single request. The model has no other information about what a tool does besides this — no source code, no comments, nothing. A vague description is exactly as harmful as a vague instruction in the system prompt, because that's precisely what it is.

Small local models (3–4B parameters) are far less forgiving of ambiguity than larger cloud models. A cloud-scale model might correctly guess intent from a terse description; a small model is more likely to skip the tool, invent arguments, or pick the wrong one among several similar-sounding tools.

## Design rules that matter for small models

| Rule | Why |
|---|---|
| Few tools | Every tool adds to the prompt and adds a chance the model picks wrong. Five tools is already a meaningful cognitive load for a 4B model. |
| Few, required parameters | Optional parameters and enum lists invite small models to invent plausible-but-wrong values. |
| Description says *when*, not just *what* | "Searches notes" tells the model nothing about when to use it versus a second read-only tool. "Use this first for any question about the user's notes" does. |
| English descriptions, even if answers are in another language | Small models are typically better at tool-calling in English regardless of the conversation's language. |
| Return results with clear structure and source | The model should be able to cite exactly where something came from. |
| Return errors as text, never let an exception escape | Covered in Lesson 1 — still applies, doubly so with unreliable argument generation from small models. |
| Validate everything inside the tool | A tool's arguments come from the model — treat them exactly as untrusted as user input, because they functionally are. |

## What you'll build

- No new tools this lesson — instead, you'll deliberately degrade and then repair your existing tool descriptions, measuring the model's behavior at each step.
