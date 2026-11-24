# Module 5, Lesson 15 — Review Checklist

## Files

- [ ] `NOTES.md` is written by the user, and the agent never edits it.
- [ ] `HANDOFF.md` is written by the agent, with the five fixed sections.
- [ ] Each file's size and loading rules follow the memory limit from Lesson 12.

## Notes as data

- [ ] A note that asks the agent to override the rules does not change the outcome.
- [ ] A note that asks the agent to hide its actions does not stop the prompts from showing.
- [ ] Secrets in the notes trigger the secret check.

## Handoff

- [ ] Lists in the handoff are built from the session record and the checkpoints.
- [ ] Each claim under Done names a file, and each check names a command and an exit code.
- [ ] A claim that the record does not support is marked unverified, with a warning.

## Staleness

- [ ] Stale notes or a stale handoff produce a single warning at session start.
- [ ] The warning says the user decides, and does not block the session.

## Notifications

- [ ] A waiting gate produces one notification after the set time, and not a repeated one.
- [ ] A notification shows the kind of action and not its content.
- [ ] Sound and desktop notifications are off by default.

## Reflection

- [ ] I tried a note that tried to override a rule, and recorded the outcome.
- [ ] I can say why a handoff built from the record is more trustworthy than one built from memory.
