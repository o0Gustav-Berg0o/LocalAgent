# Module 5, Lesson 14 — Review Checklist

## Modes

- [ ] The mode is held by the session, not the config, and switching it changes no permission rule.
- [ ] The status line shows the mode as a word at all times.
- [ ] A switch during a streaming answer takes effect after the answer.

## Plan

- [ ] The plan is a Markdown checklist, parsed with exactly the three rules from the README.
- [ ] Duplicate and missing step numbers are refused with an error.
- [ ] A plan the user has not approved never runs a step.

## Gates and checkpoints

- [ ] Each step has a gate before it, with yes, skip, and stop.
- [ ] Each step's actions still go through the permission rules.
- [ ] Each completed step has a checkpoint, and the plan records its identifier.
- [ ] Marking a step done is an atomic write, and changes only that marker.

## Editing and rollback

- [ ] The plan is read at each gate, and an edit to a future step is respected.
- [ ] The agent never changes a step marked done.
- [ ] A rollback refuses if files changed since the checkpoint, and lists them.

## Stuck and absent

- [ ] A step over the tool-call limit stops, and is marked failed with its reason.
- [ ] A gate with no answer waits, and does not proceed.
- [ ] A failed step stops the plan at the failure.

## Reflection

- [ ] I can say which mode I would give a student for a first task, and why.
- [ ] I can explain why code mode is not less safe than cowork mode.
