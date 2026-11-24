# Module 2, Lesson 5 — Review Checklist

## Paths

- [ ] Every tool resolves the requested path before checking it against the root.
- [ ] The check requires the root followed by a separator, and is case-insensitive on Windows.
- [ ] The escape cases in Exercise 1 are refused, with a reason the model can read.

## Tools

- [ ] `list_files`, `read_file`, and `search_text` are registered with snake_case names.
- [ ] Each tool returns a result in a compact, plain format with no decoration.
- [ ] A missing file or folder returns a message with a hint, not an exception.

## Limits

- [ ] `read_file` limits output by lines and by bytes, and reports the total line count when it cuts.
- [ ] `list_files` limits entries and says how many were not shown.
- [ ] `search_text` limits matches, enforces a minimum search length, and reports cuts.
- [ ] Every limit is a named value, and the ones that users may change are in the config.

## Skips and binaries

- [ ] Build and tool folders in the skip list are never read.
- [ ] A binary file is detected by its content, and refused by both `read_file` and `list_files`.

## Descriptions

- [ ] Each description says what the tool returns, when to use it, and when not to.
- [ ] Each description is under about 60 words.
- [ ] I have run the three choice questions, and each picked the expected tool.

## Reflection

- [ ] I can state the token cost of each tool's output, with a number from my own measurement.
- [ ] I can name one path-escape case that the tests do not cover, and say how I would test it.
