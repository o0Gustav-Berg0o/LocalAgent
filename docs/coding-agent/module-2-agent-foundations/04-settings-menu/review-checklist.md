# Module 2, Lesson 4 — Review Checklist

## Settings

- [ ] The settings table lists each user-facing setting with allowed values, refusal message, dependency, and "applies when".
- [ ] Internal fields such as the file version are not in the menu.
- [ ] Each setting that needs a restart or reindex shows the right message after a save.

## Effort

- [ ] Effort is one enum, mapped to limits in one table in code.
- [ ] Low, medium, and high change the answer length and the tool-call limit, and I have observed the difference.
- [ ] An unknown effort value in the config falls back to medium with a warning.

## Validation and save

- [ ] Every row of the settings table has a refusal path, tested with a bad value.
- [ ] A dependency violation names both values in its message.
- [ ] A save writes a temporary file and moves it into place, and leaves no `.tmp` file behind.
- [ ] A failed save keeps the old value on screen and leaves the file unchanged.

## Menu

- [ ] Up, down, Enter, and Esc work, and the marker never leaves the list.
- [ ] Esc with a pending change asks, and each answer does what it says.
- [ ] Key reading happens in one place.

## Terminals

- [ ] With `NO_COLOR` set, there is no colour code and the marker is still visible.
- [ ] The plain numbered fallback can change, validate, and save every setting.

## Reflection

- [ ] I can explain the difference between a setting, session state, and a mode, with an example of each.
- [ ] I can say which effort level I would give a new user, and why.
