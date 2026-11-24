# Module 2, Lesson 3 — Review Checklist

## The seam

- [ ] `AgentLoop.cs` contains no reference to the server client library.
- [ ] The loop depends on `IChatProvider`, and the fake provider can replace the real one.
- [ ] The app answers a question with `--fake` and no server running.

## Config

- [ ] Every value marked "move to config" in Exercise 0 is read from the config file.
- [ ] Every value marked "delete (duplicate)" is gone. A search for each port number finds only the intended copies.
- [ ] The design table in `config-design.md` has a type, default, rule, and invalid-value action for every field.
- [ ] The "fields we will not store" section exists, and it lists at least one secret I chose not to store.

## Validation

- [ ] Each invalid test file produces the warning from the design table, and the app starts with the default.
- [ ] Unknown fields are ignored without a crash.
- [ ] The validator never overwrites the file on disk.
- [ ] The config is written through a temporary file and renamed, so a crash during a write does not leave a broken file.

## Context

- [ ] The startup line shows both the app's and the server's context size.
- [ ] A mismatch produces a warning that names the fix.

## Errors

- [ ] A failed connection shows the endpoint, the kind of failure, and one line of advice.
- [ ] I have checked each kind of failure in Exercise 7 (refused, timeout, not found).

## Reflection

- [ ] I can explain why a value is a setting or a constant, for at least three values.
- [ ] I can say what changes in the config if the chat model moves to another machine.
