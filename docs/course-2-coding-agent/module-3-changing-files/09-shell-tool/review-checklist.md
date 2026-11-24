# Module 3, Lesson 9 — Review Checklist

## Allowlist

- [ ] The allowlist is in the config, validated on load, and refuses entries whose program contains a path.
- [ ] Each pattern matches whole arguments, and a near-miss is refused.
- [ ] I can explain, in a sentence, what each allowlist entry permits.

## Running

- [ ] Commands run from an argument list, with `UseShellExecute` off.
- [ ] The working directory is inside the project root, checked by the path rule.
- [ ] The environment is cleared, and only the listed variables are copied.
- [ ] Output is read as UTF-8, and Swedish letters are correct in a test.

## Timeout and cancel

- [ ] A timeout stops the whole process tree, and the result says so.
- [ ] Esc stops the whole process tree, and the result says cancelled.
- [ ] I checked with a process list that no child remains.

## Results

- [ ] The result holds the exit code, the duration, and the output.
- [ ] Long output keeps its end, and states how much was cut.
- [ ] The environment never appears in a result.

## Permissions

- [ ] Allowed commands run without a prompt, and their output appears in the trace.
- [ ] Unlisted commands ask, with the full command shown.
- [ ] A denied command does not run, and the model is told why.

## Limits

- [ ] The count of shell calls is limited per question, and the limit is stated to the model.

## Reflection

- [ ] I can name the allowlist entry I considered most dangerous, and what I decided.
- [ ] I can explain why `UseShellExecute = false` matters.
