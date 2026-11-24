# Module 1, Lesson 1 — Review Checklist

Use this list before you mark the lesson complete. Each item is something you can check, not something you believe.

## Setup and settings

- [ ] I can state, with one example each, what belongs in setup and what belongs in settings.
- [ ] The wizard never downloads from `/settings`.
- [ ] Re-running `/setup` shows the current value of every question, and Enter keeps it.

## Scanning

- [ ] The scan uses only read-only commands and does not need administrator rights.
- [ ] Every external command runs with a timeout, and a timeout does not stop the wizard.
- [ ] A missing `nvidia-smi` is a normal result, and the wizard continues.
- [ ] The scanner returns `null` for any fact it cannot read. It never invents a value.
- [ ] I can explain why `AdapterRAM` cannot be trusted above 4 GB, and what the wizard does about it.

## Suggestions

- [ ] The suggestion function is pure: the same input always gives the same output.
- [ ] Rules are checked in order, and the last rule is a safe default.
- [ ] A lower-bound VRAM value produces a lower class, with a reason that says so.
- [ ] Every suggestion includes the estimate warning.

## Config file

- [ ] `settings.json` holds only non-secret values, and I have searched it to confirm.
- [ ] The file has a `version` field, and unknown fields are ignored when reading.
- [ ] An invalid value produces a warning and a default, and the file is not deleted.
- [ ] The file is found next to the executable, not by the current working directory.

## Installation guidance

- [ ] The guide only prints text. It starts no process and writes no file.
- [ ] The printed build matches the detected GPU, and the guide says to choose it from the official release page.
- [ ] Every path in a printed command is quoted, and tested with a folder that has a space.

## Downloads

- [ ] Consent shows the source, the file name, the size, and the destination.
- [ ] Only the exact word `yes` continues. Enter, `y`, and closed input all cancel.
- [ ] The download writes to a `.part` file, and renames it only after success.
- [ ] A size mismatch deletes the `.part` file and reports the error.
- [ ] An existing file with the target name is never replaced silently.
- [ ] I have a written decision on what happens to a `.part` file left by an interrupted run.

## Wiring and checks

- [ ] A first start with no config runs the wizard, and a second start does not.
- [ ] Cancelling `/setup` halfway leaves the config file unchanged.
- [ ] Both endpoints are checked at the end of setup, and each failure has a one-line reason and a suggested fix.

## Edge cases

- [ ] I ran the edge-case table in Exercise 9 and recorded a result for every row.
- [ ] Any unexpected result has been fixed, and the fix is written down.

## Reflection

- [ ] I wrote answers to the four Observe questions, with cases rather than feelings.
- [ ] I can name three assumptions in my wizard that would break on another person's machine.
