# Module 2, Lesson 4 — Exercises: Settings and the Menu

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state, with Lesson 3's config loading and validation working.
- Start the chat `llama-server`.
- Test in the terminal you normally use. Also test once in a second terminal, so you know where the menu's behaviour differs.

---

## Exercise 0 — List the settings

**Goal:** a complete table of user-facing settings, before you write any menu code.

**Do**

1. Open your config design from Lesson 3 (`config-design.md`). Copy every field that a user should be able to change into a new table.
2. For each field, add these columns:
   - **Allowed values or range**
   - **Refusal message** (the sentence shown when a value is refused)
   - **Depends on** (another field, or "nothing")
   - **Applies when** ("next request", "restart", or "next reindex")
3. Leave out fields that are internal, such as the file version.

**Read this carefully.** The "applies when" column is the one students skip, and the one users complain about most. A setting that applies only after a restart must say so next to the value, not in the README.

**Checkpoint:** the table has at least five settings, and every row has all four columns filled in.

---

## Exercise 1 — Define effort

**Goal:** the effort levels map to concrete limits, and the menu shows the mapping.

**Do**

1. Create `Settings/EffortLevel.cs` with an enum (`Low`, `Medium`, `High`) and one method that returns the limits for each level: maximum output tokens and maximum tool calls per question. Use the values from the README (512 and 2, 2048 and 4, 4096 and 8), in one place.
2. Make the loop read these limits from the chosen effort, instead of from its own constants. The max tool calls replaces the iteration limit from the RAG course for this purpose.
3. Store the chosen effort in the config as a text value (`"low"`, `"medium"`, `"high"`). Parse it on load, and fall back to `medium` with a warning if the text is unknown.

Here is the shape of the mapping. Keep it as a small, readable table in code:

```csharp
public static class EffortMap
{
    // The only place where effort is turned into numbers.
    // Changing these numbers changes what every user gets for that level.
    public static (int MaxOutputTokens, int MaxToolCalls) Limits(EffortLevel level) => level switch
    {
        EffortLevel.Low    => (512, 2),
        EffortLevel.Medium => (2048, 4),
        EffortLevel.High   => (4096, 8),
        _ => throw new ArgumentOutOfRangeException(nameof(level))
    };
}
```

**Read this carefully.** The `throw` in the last arm can only happen if someone adds a new level and forgets this table. That is a programming error, not a user error, so an exception is correct here. User input is handled in the parser, which never reaches this method with an unknown value.

**Checkpoint**
- Set effort to `low` in the config, ask a question that needs three searches, and check that the trace stops after two tool calls.
- Set it to `high` and ask the same question. The search count may rise. Note what you see.

---

## Exercise 2 — Validate before save

**Goal:** a value that fails validation is never written, and the user is told why.

**Do**

1. Create `Settings/SettingsValidator.cs`. It takes the setting name and the proposed value, and returns either "accepted" or "refused, because …". It does not write anything.
2. Write one rule per row of the table from Exercise 0. For the dependency rule, the validator must also look at the current value of the other setting, and say so in the message.
3. Write the refusal messages so that each one tells the user the valid range or the value they need to change. "Invalid" alone is not a message.

**Example of a dependency message:** "contextSize 16384 is larger than the server's context size 8192. Lower contextSize, or start llama-server with a larger context."

**Checkpoint:** for each row of the table, a value that is too large, too small, of the wrong type, and violating a dependency is refused, with the message you wrote in Exercise 0.

---

## Exercise 3 — Atomic save

**Goal:** a save either completes or leaves the old file exactly as it was.

**Do**

1. Create `Settings/SettingsStore.cs` with a `SaveAsync(AppConfig config)` method. It writes the JSON to `settings.json.tmp` next to the real file, then moves it over the real file in one step. If the write fails, the temporary file is deleted, and the real file is untouched.
2. Make the menu call `SaveAsync` only after the validator accepts the value. Keep the old value in memory until the save returns successfully.
3. Simulate a failed save: make the folder read-only for the save, or hold the file open in another program. The menu must show the failure, and the screen must still show the old value.

**Read this carefully.** The order matters. Write the temporary file, then move it. Deleting the real file first and then writing would leave a window in which no settings file exists at all.

**Checkpoint**
- A normal save changes the file, and no `.tmp` file is left behind.
- A failed save shows an error, the old value stays on the screen, and the file is unchanged.

---

## Exercise 4 — The menu, step by step

**Goal:** a navigable menu with a clear current value on each row.

**Do**

1. Create `Settings/SettingsMenu.cs`. It shows one row per setting: the name, the current value, and a short note. Use the table from Exercise 0 for the content.
2. Make the selected row marked with a character such as `>`. Use colour only in addition to the marker, and only if colour is allowed (Exercise 6).
3. Read the keys with a method that returns the key pressed. Map up and down to moving the selection, Enter to editing the selected row, and Esc to going back.
4. When a row is in edit mode, read a line of text, parse it, validate it, and save it. Then return to the list with the new value shown.

**Read this carefully.** Read key presses in one place. If the menu reads keys in two methods, a key pressed during one can be lost in the other.

**Checkpoint**
- Up and down move the marker, and the marker never leaves the list.
- Enter on a number setting lets you type a new value. Enter on the effort setting cycles through the three levels, or opens a choice.
- A refused value shows its message and stays in edit mode.

---

## Exercise 5 — Quit with unsaved changes

**Goal:** the user never loses a confirmed change, and is asked before losing a pending one.

**Background:** a change is "confirmed" when it has been saved. A change is "pending" when the user has typed it but not pressed Enter to confirm. Only pending changes can be lost, and the menu must ask before discarding them.

**Do**

1. Track pending changes: the row and the new value, before it is saved.
2. When the user presses Esc with pending changes, show a question: save, discard, or keep editing. Read the answer as one key.
3. Make sure that a discard does not write anything to the file, and a save writes exactly the pending values.

**Checkpoint**
- Esc with no pending changes returns immediately.
- Esc with a pending change asks, and each of the three answers does what it says.

---

## Exercise 6 — Plain output and `NO_COLOR`

**Goal:** the menu works without colour and without arrow keys.

**Do**

1. Check the environment variable `NO_COLOR` at start. If it is set to any value, turn colour off for the whole app, not only for the menu.
2. Add a fallback for key reading: if the terminal does not support the key-reading method, switch to a numbered list where the user types a row number and then a value.
3. Test both modes: once with `NO_COLOR` set, and once with the fallback forced (for example with a setting `"menuMode": "plain"` in the config).

**Read this carefully.** The fallback is a real mode, not a broken version. Test it as carefully as the arrow-key menu, and make sure the same validator and the same atomic save are used.

**Checkpoint**
- With `NO_COLOR` set, the output has no colour codes, and the selected row is still marked.
- In plain mode, the same settings can be changed, validated, and saved.

---

## Exercise 7 — Show what applies when

**Goal:** each setting shows whether it takes effect now, after the next request, or after a restart or reindex.

**Do**

1. Add the "applies when" value from Exercise 0 to each row of the menu, as a short label.
2. When a setting that needs a restart is saved, show a message: "Saved. Restart the app for this to apply."
3. When a setting that needs a reindex is saved, show: "Saved. Run /reindex to apply."

**Observe**
- Which setting did you put in the "next request" group, and which would have surprised a user if you had put it there? Justify each choice.

**Checkpoint:** every setting has a label, and changing one that needs a restart or reindex shows the right message.

---

## Exercise 8 — Test the menu

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Change effort | set to high, ask a question | the limits from the map apply |
| Refused value | type a number outside the range | message names the range, value kept |
| Dependency violated | raise contextSize above the server value | message names both values |
| Failed save | make the folder read-only | error shown, old value kept, file unchanged |
| Esc with pending change | change a value, press Esc, choose discard | file unchanged, value reverted on screen |
| `NO_COLOR` set | run with the variable set | plain output, marker still visible |
| Plain mode | force the numbered fallback | all settings can be changed |
| Esc during an answer | open the menu while the model streams | the menu waits, or the change applies to the next request, as documented |

**Checkpoint:** every row has a written result, and every surprise is fixed or written down with a reason.

---

## Observe (whole lesson)

1. Which setting did you find hardest to explain to a user in one sentence? Why?
2. Which effort level would you give a new user, and why? Would that answer change for an experienced user?
3. A user changes the context size and the answer gets shorter, without any error. What could cause that, and what should the menu have shown?

---

## Discuss

- Why is effort defined in code, and not in the config file? Who should be able to change the numbers behind "high"?
- A menu that cannot lose a confirmed change is a promise. What does the user have to trust to believe it?
- The menu exposes settings but not state. Is there a case where a user would expect a state value to appear there? Which one?

---

## Stretch

- Add a `/settings reset` command that restores the defaults after a confirmation. What must it leave untouched?
- Add a search box to the menu for long setting lists. What is the simplest version that works?
- Show a short history of setting changes in a log file. What should the log never contain?

---

## Time check

If you are past the planned time, finish Exercises 2 and 3. Validation and atomic save protect the file. The menu's visual polish can be finished later.
