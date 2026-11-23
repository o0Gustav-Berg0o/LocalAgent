# Lesson 10 — Review Checklist

- [ ] You recorded `/eval` and `/search-hybrid` results before moving any code, and checked that two `/eval` runs gave the same result.
- [ ] You moved code in small steps, and ran `/eval` after each step.
- [ ] The shared code lives in a class library, and the console app references it, not the reverse.
- [ ] `NoteTool` stayed in the app, and the library has no knowledge of tools or the model.
- [ ] One factory method builds the search object, and the console app calls it instead of repeating the setup.
- [ ] `/eval` gives the same PASS/FAIL list, question by question, as before the refactor.
- [ ] You can explain each public member of the library and why it is public.
- [ ] You did not fix any bug during the move. If you found one, you wrote it down for a separate change.

## Self-check questions

1. Why did this refactor happen in Lesson 10 and not in Lesson 2? Use the project's history in your answer.
2. Name one piece of code that moved to the library and one that stayed in the console app, and say why.
3. If `/eval` shows one question flipping from PASS to FAIL after a move, what would you check first, and what would you do before trying the next move?
4. What would break in Lesson 11 if the factory method still depended on the console app's `Program.cs`?
