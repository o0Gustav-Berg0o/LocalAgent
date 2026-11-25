# Module 2, Lesson 6 — Exercises: Permissions

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/Permissions/` for the modes, the session rules, and a `PermissionsCommand`. `Rule`, `PermissionEngine`, and `DecisionLog` go in `Shared/Permissions/`, and `AskPrompt` in `Shared/Prompts/`, because the loop, the shell tool, hooks, memory, and subagents use them.
- Start from a clean git state, with Lesson 5's read tools working.
- Use the sandbox project from Lesson 5.
- Start the chat `llama-server`.
- For this lesson, add a test tool that writes nothing and only returns its input. It stands in for a write tool until Lesson 7. Name it `test_write`, and give it one string argument. Call it in the exercises when you need a "write" request.

---

## Exercise 0 — See what the model can do today

**Goal:** know which tools run without any check, before adding checks.

**Do**

1. List every tool registered in `Program.cs`, with its name.
2. For each tool, write whether it can change anything on disk, can run a program, or can read outside the project.
3. Ask the model to read a file outside the sandbox (for example a file in your user folder). Note whether it was read.

**Observe**
- Which tool would be dangerous if the model used it wrongly? Which would be dangerous if a document told the model to use it?

**Checkpoint:** you have the list, and you have a note on the outside-read test.

---

## Exercise 1 — The evaluation function

**Goal:** one function takes a request and the rules, and returns the outcome and the rule that decided it. The order is fixed and tested.

**Do**

1. Create `Shared/Permissions/Rule.cs` with a record: tool name (or `*`), target pattern (or `*`), outcome, and an optional note.
2. Create `Shared/Permissions/PermissionEngine.cs` with a method `Evaluate(string tool, string target, IReadOnlyList<Rule> rules)`. It returns the outcome and the matching rule, or the default with no rule.
3. Implement the order from the README: check all deny rules first. If one matches, return deny. Then allow rules. Then ask rules. Then the default, which is ask.
4. Write the matcher for targets as a function that takes a pattern and a resolved path. Support `*` for any and `**` for any depth inside a folder. Keep the pattern language small, and document it.

Here is the skeleton. The matcher is the part to write carefully:

```csharp
namespace LocalAgent.Shared.Permissions;

public enum Outcome { Allow, Ask, Deny }

// Tool is a tool name or "*". Target is a pattern (step 4). Note is shown to the user and the model.
public record Rule(string Tool, string Target, Outcome Outcome, string? Note = null);

public static class PermissionEngine
{
    public static (Outcome Outcome, Rule? Rule) Evaluate(
        string tool, string resolvedTarget, IReadOnlyList<Rule> rules)
    {
        // 1. Deny rules win. Check them all before anything else.
        // 2. Then allow rules.
        // 3. Then ask rules.
        // 4. Otherwise, the default: ask, with no rule.
        throw new NotImplementedException();
    }

    // "*" matches anything in one folder level, "**" matches any depth.
    // Compare case-insensitively on Windows. Document the pattern language in this comment.
    public static bool Matches(string pattern, string resolvedPath)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The function must check the target that was **resolved** (Lesson 5), not the text the model sent. Pass the resolved path in. If you pass the raw text, every test below will look right and still be wrong.

**Checkpoint:** write the table below as tests or as a temporary console check. Every row must match.

| Rules (in this order) | Request | Expected outcome |
|---|---|---|
| allow read_file `**` | read_file `src\a.cs` | allow |
| deny read_file `**\.env` and allow read_file `**` | read_file `src\.env` | deny (deny wins) |
| nothing | write_file `a.txt` | ask (default) |
| allow write_file `**` and deny write_file `**` | write_file `a.txt` | deny (deny wins) |
| ask write_file `docs\**` | write_file `src\a.cs` | ask (default, because no match) |

---

## Exercise 2 — The three modes as presets

**Goal:** the modes are data: named rule sets, not separate code paths.

**Do**

1. Create `Features/Permissions/Modes.cs` with a dictionary from mode name to rule list: `read-only`, `ask`, `trusted`. Use the table in the README (section 3.5) as the starting point.
2. Store the active mode in the config. Load it at start, and fall back to `ask` with a warning if the name is unknown.
3. Make the engine receive the rules of the active mode, not a mode name. The engine does not know modes exist.

Start from this shell:

```csharp
namespace LocalAgent.Features.Permissions;

public static class Modes
{
    public const string Default = "ask";

    // Mode name to rules. Fill each list from the table in README section 3.5.
    public static readonly IReadOnlyDictionary<string, IReadOnlyList<Rule>> Presets =
        new Dictionary<string, IReadOnlyList<Rule>>(StringComparer.OrdinalIgnoreCase)
        {
            ["read-only"] = [ /* rules from the table */ ],
            ["ask"]       = [ /* rules from the table */ ],
            ["trusted"]   = [ /* rules from the table */ ],
        };

    // An unknown or missing name gives the rules of Default and a warning (step 2).
    public static (IReadOnlyList<Rule> Rules, string? Warning) For(string? modeName)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Keeping the engine free of mode names means a new mode is a data change, not a code change. It also means a bug in the engine affects every mode in the same way, which is the point.

**Checkpoint**
- Set the mode to `read-only` in the config. A write request is denied, and the reason names the mode.
- Set it to `ask`. The same write request is asked.

---

## Exercise 3 — One place for the check

**Goal:** every tool call goes through the check, and no tool can be called around it.

**Do**

1. Find every place in the loop where a tool is invoked. There should be one. If there is more than one, merge them.
2. Before the invoke, call the engine with the tool name and the resolved target. The target is the path for file tools, and the whole argument text for other tools in this lesson.
3. Act on the outcome:
   - allow: invoke the tool,
   - deny: do not invoke it; return a result to the model that says "denied by permission rule: <rule note or default>",
   - ask: pause, show the prompt (Exercise 4), and act on the answer.
4. Make sure a denied call returns a tool result, so the model's conversation stays valid (each call needs a result, Lesson 2).

Start from this shell, in `AgentLoop`:

```csharp
// The one place where a tool runs. It always returns a result for the model,
// so a denied or refused call still gets its tool message (step 4).
private Task<string> InvokeCheckedAsync(FunctionCallContent call, AIFunction tool, CancellationToken cancellationToken)
{
    // 1. The target: the resolved path for a file tool, the argument text for any other tool.
    // 2. PermissionEngine.Evaluate(call.Name, target, activeRules)
    // 3. Allow: invoke. Deny: "denied by permission rule: <note or default>". Ask: AskPrompt (Exercise 4).
    throw new NotImplementedException(); // students write this
}
```

**Read this carefully.** A denied call must still produce a tool message. If you skip it, the next request to the model may be rejected or confuse it. This is a common bug.

**Checkpoint**
- A read inside the sandbox is allowed and runs.
- A read of a file outside the sandbox is denied, and the model's answer explains that it was not allowed to read it.
- A denied call appears in the trace as a tool result, not as a crash.

---

## Exercise 4 — The ask prompt

**Goal:** a question that lets the user decide with the facts in front of them.

**Do**

1. Write `Shared/Prompts/AskPrompt.cs`. It shows: the tool in plain words, the full resolved target, a short summary of what will happen (for the test tool, its input), and the choices.
2. The choices for this lesson are: `yes` (this request only), `no`, and `always` (for this session). Read the answer with a method that accepts only these exact words, and asks again on anything else.
3. Show the same prompt format for every tool, so the user learns it once.

Start from this shell:

```csharp
namespace LocalAgent.Shared.Prompts;

public enum AskAnswer { Yes, No, Always }

public static class AskPrompt
{
    // Prints the request in the format below and reads one of the exact words: yes, no, always.
    // "always" is accepted only when offerAlways is true. Anything else asks again.
    // Closed input (ReadLine returns null) counts as No.
    public static AskAnswer Ask(string action, string target, string change, bool offerAlways,
        TextReader input, TextWriter output)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

Example of the printed text. Adapt the wording:

```text
The assistant wants to: write a file
Target: C:\Repos\sandbox\notes\todo.txt
Change: create a new file (12 lines)

Allow?  yes = this time   no = refuse   always = for this session  >
```

**Read this carefully.** "Always" is offered only where it makes sense. For a test tool with a fixed input, "always for this session" is reasonable. For a shell command with arguments that vary, it may not be (Lesson 9 handles that).

**Checkpoint:** a request in ask mode shows the prompt, the three answers work, and any other text is asked again.

---

## Exercise 5 — Always, with a scope

**Goal:** "always" creates a rule with a scope, and the user can see and remove it.

**Do**

1. When the user answers `always`, add an allow rule for this exact tool and this exact target, to the session rules. The session rules are kept in memory and cleared when the app closes.
2. Create a `/permissions` command that lists the active rules, grouped by scope: mode rules, session rules, and project rules (project rules come in Lesson 12, so show the group as empty).
3. Create a way to remove a session rule, from `/permissions`. Use the number of the rule.
4. Make sure a session rule never overrides a deny rule from the mode. Deny still wins, by the order in Exercise 1.

Start from this shell. The engine receives the mode rules and the session rules in one list:

```csharp
namespace LocalAgent.Features.Permissions;

// In memory only. Cleared when the app closes.
public sealed class SessionRules
{
    public IReadOnlyList<Rule> Rules => throw new NotImplementedException(); // students write this

    // An allow rule for this tool and this exact resolved target, nothing wider (step 1).
    public void AddExact(string tool, string resolvedTarget)
    {
        throw new NotImplementedException(); // students write this
    }

    // number is 1-based, as /permissions shows it. Returns false for an unknown number.
    public bool Remove(int number)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The exact-target rule is the safe default. A rule that allows "all writes" after one approval is far more dangerous. Start narrow, and widen only with a deliberate choice that the user sees.

**Checkpoint**
- Answer `always` to a test write. The next identical request runs without asking.
- `/permissions` shows the rule. Removing it makes the next request ask again.
- In read-only mode, a session `always` rule does not allow the write, because the mode's deny wins.

---

## Exercise 6 — The decision log

**Goal:** every decision is recorded, so the user can see what happened and why.

**Do**

1. Create `Shared/Permissions/DecisionLog.cs` that appends one line per decision to a log file next to the config. Each line holds: time, tool, target, outcome, the rule that decided it (or "default"), and whether the user was asked and their answer.
2. The log must not contain file contents or tool arguments that might be large or secret. Log the target and the summary only.
3. Add `/permissions log` to show the last twenty lines.

Start from this shell:

```csharp
namespace LocalAgent.Shared.Permissions;

// What goes in the log, and what never does: (write the decision here, step 2).
public sealed class DecisionLog(string logPath)
{
    // One line per decision. answer is null when the user was not asked.
    // Never throws: returns the error text if the line could not be written, or null.
    // The decision applies either way (Exercise 7, the locked log file).
    public string? Append(string tool, string target, Outcome outcome, Rule? rule, string? answer)
    {
        throw new NotImplementedException(); // students write this
    }

    public IReadOnlyList<string> Last(int count)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** A log that records file contents becomes a second copy of the user's data in a place they did not expect. Decide what goes in, and write the decision in a comment at the top of the log class.

**Checkpoint**
- Each request, allowed or not, adds one line.
- The log has no file contents.
- `/permissions log` shows the last lines.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Deny and allow both match | rules from Exercise 1, row 2 | deny |
| Mode switched while an ask is waiting | start a write in ask mode, switch to read-only in another step | the waiting request is refused, not approved |
| Always, then mode change to read-only | answer always, then change mode | the write is denied |
| Unknown answer at the prompt | type `maybe` | asked again |
| Closed input at the prompt | redirect input from an empty file | the request is refused, no crash |
| Path with a short name | request a path using `PROGRA~1` | resolved, and matched against the rule |
| Many requests in one turn | a question that causes three writes | three prompts, each answered once |
| Log file locked by another program | open the log in an editor, then make a request | the decision still applies; the log error is reported |

**Checkpoint:** every row has a written result, and every surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which outcome did you find easiest to get wrong: allow, ask, or deny? Why?
2. Give one example where "always" would be the right answer for the user, and one where it would be dangerous. What is the difference?
3. A user says the prompts are too frequent. What would you change first: the rules, the mode, or the prompt? Why?

---

## Discuss

- The check runs in code, so it cannot be argued with. What can it not protect against? (Hint: a bad rule the user accepted.)
- Why is the default "ask" and not "deny"? When would "deny" be the better default?
- Approval fatigue is a design problem as much as a user problem. Name two changes to the design that reduce it.

---

## Stretch

- Add a rule that asks once per hour instead of once per request, and reason about its risk.
- Add a "dry run" mode that shows what would happen for each request without doing it. What would be missing from the dry run?
- Add a signed rules file, so that a rule added by someone else is shown and needs a check. Is that worth the complexity for a local tool?

---

## Time check

If you are past the planned time, finish Exercises 1, 3, and 4. The evaluation order, the single check point, and the ask prompt are the parts the next two lessons depend on.
