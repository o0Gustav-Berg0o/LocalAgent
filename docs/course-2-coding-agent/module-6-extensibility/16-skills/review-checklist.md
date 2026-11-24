# Module 6, Lesson 16 — Review Checklist

## Format

- [ ] A skill is a folder with `SKILL.md`, a header with `name` and `description`, and a body.
- [ ] Names are lowercase with hyphens, and descriptions say when to use the skill.

## Index

- [ ] The index reads only the headers, never the bodies.
- [ ] The index costs tokens that I measured, and is smaller than the instruction set it replaces.
- [ ] A skill with a missing field is skipped with a warning.

## Loading

- [ ] `load_skill` accepts only names from the index, and refuses others with the list of valid names.
- [ ] Each skill loads at most once per session.
- [ ] A body over the size limit is cut, and the cut is stated.

## Choice

- [ ] I measured the model's choice on ten tasks, and recorded the score.
- [ ] I changed at least one description after a wrong choice, and checked the change.

## Rules

- [ ] A body that tries to override a permission rule does not change the outcome.
- [ ] A body's tool requests go through the same check as any other request.

## Overrides and limits

- [ ] A project skill overrides a user skill with the same name, and the index shows it.
- [ ] The count, size, and index limits are enforced, and each refusal names its limit.

## Reflection

- [ ] I can say how many skills my index can hold before it costs more than it saves.
- [ ] I can explain why a skill body is data and cannot change the rules.
