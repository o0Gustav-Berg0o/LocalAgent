# Module 1, Lesson 1 — Quiz

Write your own answer to each question first, then compare with the answer key at the end.

1. What is the difference between first-run setup and settings? Give one example of something that belongs in each, and say why a download belongs only in setup.
2. Why is the `AdapterRAM` value from `Win32_VideoController` unreliable for cards with more than 4 GB?
3. Why does the scanner return `null` for an unknown fact, instead of a guessed value? What goes wrong in the suggestion step if it guesses?
4. What is the purpose of downloading to a `.part` file and renaming it only when complete? Describe one failure that the size check alone would miss.
5. Why must the consent step accept only the exact word `yes`? What happens with Enter, with closed input, and with `y`?
6. Why should `settings.json` never contain an API key, even for a local-only app?
7. A user runs `/setup` and presses Enter at every question. What should be written to the file, and why is it important that Enter keeps the existing values?

---

## Answer key

1. **Setup** runs once and asks what the machine can run and what the user wants. **Settings** changes one value at a time later. A download belongs only in setup because a settings change could then start a multi-gigabyte download without the user noticing. Setup keeps every download behind its own confirmation.
2. The field is a 32-bit number, so it cannot store more than about 4 GB. A card with 8 or 16 GB may report a value near 4 GB or a capped value. Use `nvidia-smi` for NVIDIA cards and treat other values as a lower bound.
3. A guessed value looks like a real fact to the suggestion rules, so the wizard can suggest a model that does not fit. An unknown fact should fall to the safe default, and the reason should say the value was unknown.
4. A download that stops partway leaves a file that looks complete under its real name. Other programs may open it and fail in confusing ways. A size check catches a truncated file, but not a file of the right size with wrong content. A checksum catches that case, if the publisher provides one.
5. An exact `yes` prevents accidental downloads. Enter gives an empty string, so it cancels. Closed input gives `null`, which must be handled and also cancels. `y` is a common habit, but accepting it makes the rule ambiguous, so the wizard asks for the exact word it names.
6. The config file is plain text, can be copied, shared, or committed by mistake, and is read by any program with access to the folder. A local-only app has no reason to hold a key, so storing one creates a risk without a benefit.
7. The file should hold the same values it held before, plus any change the user made. Enter keeps the current values, so a user who only checks the settings does not lose anything. Writing defaults over real values would destroy the user's choices.

---

## Scoring

- 6 or 7 correct: ready for Module 1, Lesson 2.
- 4 or 5 correct: review the sections of the README that the missed questions point to, then re-do the Checkpoint in that exercise.
- Fewer than 4: repeat Exercises 2 and 6 before moving on. Those two carry the most safety and data-quality risk.
