# Lesson 5 — Quiz

Write your own answer to each question first, then check it against the answer key below.

## Questions

1. What information does a model actually have about a tool, besides its registered name, description, and parameter schema?
2. Why are small (3–4B) local models less forgiving of a vague tool description than large cloud models?
3. What should a good tool description communicate, beyond what the tool does?
4. Why should a tool's arguments be validated inside the tool itself, not trusted as given?
5. Why does the course recommend English tool descriptions even in a non-English conversation?

## Answers

1. Nothing — no source code, no comments, nothing else. The name, description, and schema are its entire picture of what a tool does.
2. Larger models can often correctly guess intent from a terse description; smaller models are more likely to skip the tool entirely, invent arguments, or pick the wrong one among similar-sounding options.
3. *When* to use it, not just what it does — e.g. "use this first for any question about the user's notes" tells the model something a bare "searches notes" doesn't, especially when choosing between similar tools.
4. Because arguments come from the model, which can invent or get them wrong — they should be treated exactly as untrusted as user input, because functionally they are.
5. Small models are typically better at tool-calling in English regardless of what language the conversation itself is in.
