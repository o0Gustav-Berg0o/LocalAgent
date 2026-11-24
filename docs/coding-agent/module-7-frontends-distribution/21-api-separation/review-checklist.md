# Module 7, Lesson 21 — Review Checklist

## Boundary

- [ ] The core contains no screen writes and no keyboard reads.
- [ ] Every mixed place from Exercise 0 has been split, or is recorded as a known exception with a reason.

## Contract

- [ ] Events and commands are typed records with no behaviour.
- [ ] Each event and command has a comment that says what it means.
- [ ] The contract has a version, and the rule for breaking changes is written in it.

## Approvals

- [ ] Every approval has an identifier, and the answer must match it.
- [ ] An answer with an unknown or old identifier is refused, and the refusal is an event.
- [ ] The core waits for the answer; the frontend only sends it.

## Frontends

- [ ] The console runs on the contract, and the answer still streams.
- [ ] A second frontend runs on the same core, and no core file changed to add it.
- [ ] A slow frontend does not slow the core beyond its bound.

## Tests

- [ ] At least two tests run the core with no console, and assert on events.
- [ ] A test answers an approval with no, and the file is unchanged.

## Versioning

- [ ] A frontend and the core agree on a version at connection.
- [ ] Adding an optional field works with an old frontend.
- [ ] A rename is refused with a message that names both versions.

## Reflection

- [ ] I can say what the core now knows that it did not know about the screen, and what the frontend now knows about the agent.
- [ ] I can name one change that would be breaking, and say why.
