# Module 4, Lesson 12 — Review Checklist

## Instructions

- [ ] `AGENT.md` is read once per session, and added to the system prompt under a heading that names its source.
- [ ] The instruction file is never written by a tool.
- [ ] A missing instruction file does not cause an error.

## Cost

- [ ] Memory is counted as its own part of the context meter, with its size in tokens.

## Agent notes

- [ ] A proposed note is shown with its section and reason, and is written only on yes or edit.
- [ ] Notes live in a separate file from the user's instructions.
- [ ] Each saved note carries a date.

## Limits and secrets

- [ ] A size limit for agent memory is enforced before a note is saved.
- [ ] A secret check refuses a note that looks like a key, and does not print the secret.
- [ ] I have written in my README that the secret check is a heuristic.

## The `/memory` view

- [ ] The view shows every memory file, with size, date, and sections.
- [ ] A git check reads status only, and never stages or commits.
- [ ] A tracked, non-empty memory file produces a warning.

## Reflection

- [ ] I can name one fact the agent remembered only because of memory, and one it still got wrong.
- [ ] I can explain why an agent note needs more care than a user's instruction file.
