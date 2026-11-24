# Module 6, Lesson 17 — Review Checklist

## Files

- [ ] A command is a file with a `description` header, an optional `arguments` header, and a template.
- [ ] A file with no description is skipped with a warning.

## Expansion

- [ ] Expansion replaces `$ARGUMENTS` only, and produces text.
- [ ] The expanded text is sent as a user message through the normal loop.
- [ ] No command calls a tool directly.
- [ ] The expanded prompt is shown to the user.

## Names and limits

- [ ] A command whose name matches a built-in is refused, with a warning.
- [ ] Each argument is limited, and a cut is stated in the expanded prompt.

## Help and autocomplete

- [ ] `/help` and autocomplete read from the same list.
- [ ] A new command file appears without restarting the app.

## Rules

- [ ] A command cannot change the mode, the settings, or the permission rules.
- [ ] A command's request for a shell command follows the allowlist.

## Reflection

- [ ] I wrote down three repeated requests and turned one into a command.
- [ ] I can explain why an expansion is text and not an action.
