# Module 2, Lesson 3 — Exercises: Provider and Config

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Shared/Providers/` for the provider interface and both providers, because the loop and later slices use them. `ConfigValidator` goes in `Shared/Config/`, and the new `AtomicFile` in `Shared/Files/`.
- Start from a clean git state, with the Lesson 2 streaming loop working.
- Start the chat `llama-server` on port 8080. Note the context size you started it with. You will need it in Exercise 6.
- Keep your Lesson 1 table of hardcoded values to hand.

---

## Exercise 0 — Find every hardcoded value

**Goal:** a complete list of values that must move into the config file. You will check the list again at the end.

**Do**

1. Search the whole app folder, excluding `bin` and `obj`, for each of these:
   - the chat endpoint (`8080`),
   - the embedding endpoint (`8081`),
   - each model name you know of,
   - the notes folder path,
   - the context size (look for `8192`, `4096`, or any number that looks like a token count),
   - any timeout (look for `TimeSpan`).
2. Write each match in a table with three columns: value, file and line, and whether it is a setting or a constant of the code.
3. Mark each entry as one of: **move to config**, **keep in code**, or **delete (duplicate)**.

**Read this carefully.** Not every number is a setting. A `0.3f` threshold is a tuning value that belongs with the search code, not in the user's config. A port number is a setting. Deciding which is which is the first design choice of this lesson. Write your reason next to each decision.

**Checkpoint:** the table has at least five entries, each with a decision and a reason.

---

## Exercise 1 — Define the provider interface

**Goal:** the loop talks to an interface. The interface is small enough to fake in a test.

**Do**

1. Create `Shared/Providers/IChatProvider.cs`. Give it one method for a streaming answer, taking the history and the tool list, and returning the stream of updates. Use the same types as the loop, so the loop does not change its shape.
2. Create `Shared/Providers/LocalLlamaProvider.cs` that implements the interface, and takes the endpoint, model name, and timeout as constructor parameters. It creates the server client inside itself.
3. Change `AgentLoop` so that its constructor takes an `IChatProvider`, not a client. Remove the server client import from `AgentLoop.cs`.
4. In `Program.cs`, construct the `LocalLlamaProvider` and pass it to the loop.

Here is the shape of the interface. Fill in the types from your project:

```csharp
namespace LocalAgent.Shared.Providers;

// The only thing the loop knows about a model server.
public interface IChatProvider
{
    // Streams the model's answer as updates. The history and tools come from the loop.
    IAsyncEnumerable<ChatResponseUpdate> StreamAsync(
        IEnumerable<ChatMessage> history,
        IList<AITool> tools,
        CancellationToken cancellationToken);
}
```

And the shape of the provider. Create the server client the same way `Program.cs` does now, then move that code here:

```csharp
using Microsoft.Extensions.AI;

namespace LocalAgent.Shared.Providers;

// The only class that knows the server client library. Keep the client in a private field.
public sealed class LocalLlamaProvider : IChatProvider
{
    public LocalLlamaProvider(string endpoint, string model, TimeSpan timeout)
    {
        throw new NotImplementedException(); // students write this
    }

    // Puts the tools in ChatOptions and returns the client's streaming sequence.
    public IAsyncEnumerable<ChatResponseUpdate> StreamAsync(
        IEnumerable<ChatMessage> history,
        IList<AITool> tools,
        CancellationToken cancellationToken)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The interface names what the loop needs, not what the server does. If you find yourself adding a parameter for a server-specific option, stop: that option belongs in the provider's constructor, not in the interface.

**Checkpoint**
- `AgentLoop.cs` contains no mention of the server client library.
- The app still answers a question, using the same server and the same settings as before.
- You can explain, in one sentence, what would change if you wanted to switch to a different server later.

---

## Exercise 2 — A fake provider for tests

**Goal:** the loop can run without a model server. This lets you test the loop's logic alone.

**Do**

1. Create `Shared/Providers/FakeProvider.cs` that implements the same interface. It returns a fixed answer, streamed as two or three pieces, and it never makes a network call.
2. Add a command-line flag `--fake` to `Program.cs`. When the flag is present, the app uses the fake provider instead of the local one.
3. Run `dotnet run -- --fake`, and ask a question. The answer should be the fixed text, and the loop should print it as before.

Start from this shell:

```csharp
using Microsoft.Extensions.AI;

namespace LocalAgent.Shared.Providers;

// Streams a fixed answer in two or three pieces. Never makes a network call.
public sealed class FakeProvider(string answer = "This answer comes from the fake provider.") : IChatProvider
{
    // One ChatResponseUpdate per piece, with the assistant role.
    // An async iterator (async IAsyncEnumerable<T> with yield return) is the shortest way.
    public IAsyncEnumerable<ChatResponseUpdate> StreamAsync(
        IEnumerable<ChatMessage> history,
        IList<AITool> tools,
        CancellationToken cancellationToken)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Observe**
- What did you test with the fake provider that you could not have tested with the real server? What did you not test?
- If the fake provider returns a tool call, does the loop run the tool? Try it.

**Checkpoint:** with `--fake`, the app answers without any server running, and the streaming display works.

---

## Exercise 3 — Design the config file

**Goal:** decide the fields, their types, their defaults, and their validation rules before you write any loader code.

**Do**

1. Create `config-design.md` in the app folder. For each setting from Exercise 0, write a row:

```text
Field            Type     Default              Valid range or format                   If invalid
chatEndpoint     string   http://localhost:8080/v1   must start with http:// or https://  warn, use default
chatModel        string   qwen3-vl-4b          non-empty, no spaces                    warn, use default
contextSize      int      8192                 512 to 131072                           warn, use default
timeoutSeconds   int      300                  10 to 3600                              warn, use default
```

2. Add a row for each other field from Exercise 0.
3. Add a section called **Fields we will not store**, and list the secrets and machine facts you chose not to put in the file, with a reason for each.
4. Add a section called **Version**, and decide what the `version` field means and what the app does when it is missing.

**Read this carefully.** The table is a contract. Every row becomes a test in Exercise 5. A field without a rule is a field that will eventually hold a value nobody expected.

**Checkpoint:** every field has a type, a default, a rule, and an action for invalid values. The "not stored" section is not empty.

---

## Exercise 4 — Read and write the config

**Goal:** the app reads its settings from `settings.json` on every start, and writes them when the user changes them.

**Do**

1. Extend the `AppConfig` record from Module 1, Lesson 1 with any field from your design that is not yet there.
2. Make sure `ConfigStore.Load()` returns defaults when the file does not exist, as in Module 1.
3. Make sure `ConfigStore.Save()` writes indented JSON, and writes to a temporary file first, then renames it. This is the same rule as the download in Module 1, applied to the config, so a crash during a write cannot leave a half-written file.

The new fields and the new `Save`:

```csharp
public record AppConfig(
    // ...every field from Module 1, Lesson 1...
    int TimeoutSeconds          // new, from config-design.md. Give it a value in Defaults too.
);

// In ConfigStore, Save becomes: serialize with the options from Module 1, then AtomicFile.Write.
```

The temporary-file rule goes in one shared helper. The settings store (Lesson 4) and the plan file (Lesson 14) reuse it, and the write tools (Lesson 7) build their extra checks around the same idea:

```csharp
// Shared/Files/AtomicFile.cs
namespace LocalAgent.Shared.Files;

public static class AtomicFile
{
    public static void Write(string path, byte[] content)
    {
        var tempPath = path + ".tmp";
        // 1. Write content to tempPath, in the same folder as path.
        // 2. File.Move(tempPath, path, overwrite: true). The replace is one step.
        // 3. On any exception: delete tempPath if it exists, then rethrow. The old file is untouched.
        throw new NotImplementedException(); // students write this
    }
}
```
4. Replace every value from Exercise 0 that is marked **move to config** with a read from the loaded config.
5. Delete the copies marked **delete (duplicate)**.

**Checkpoint**
- Change `chatEndpoint` in `settings.json` to a different port, start the app, and confirm the new port is used (the connection fails, and the error names the new port).
- Put the original value back. The app works again.
- Search the app folder again for each hardcoded value. The only matches left are the ones marked **keep in code**.

---

## Exercise 5 — Validate every field

**Goal:** every rule in your design table is enforced by code, in one place, and tested.

**Do**

1. Create `Shared/Config/ConfigValidator.cs`. It takes the loaded config and returns two things: a corrected config, and a list of warnings. Each warning names the field, the value that was rejected, and the default that was used. Move the rules from `ConfigStore.Validate` (Module 1) here, so there is one place for them:

```csharp
namespace LocalAgent.Shared.Config;

public static class ConfigValidator
{
    // One rule per row of config-design.md. Never throws, and never writes the file.
    // The corrected config is used in memory only, until the user saves.
    public static (AppConfig Config, IReadOnlyList<string> Warnings) Validate(AppConfig config)
    {
        var warnings = new List<string>();
        // One block per field, in the order of the design table (step 2).
        throw new NotImplementedException(); // students write this
    }
}
```
2. Write one rule per field, following your design table. A rule looks like this:

```csharp
// Each rule either accepts the value, or returns a fallback and a warning.
// The validator never throws; the app must start with any file content.
if (!Uri.TryCreate(config.ChatEndpoint, UriKind.Absolute, out var uri)
    || (uri.Scheme != "http" && uri.Scheme != "https"))
{
    warnings.Add($"chatEndpoint '{config.ChatEndpoint}' is not an http(s) address; using the default.");
    config = config with { ChatEndpoint = AppConfig.Defaults.ChatEndpoint };
}
```

3. Call the validator right after loading, and print the warnings before the app starts.
4. Write a test file, or a list of files, that you use to check the validator. Make one file per invalid case (missing brace, wrong type, out-of-range number, unknown field). A file for each rule in the design table is the goal.

**Read this carefully.** The validator must accept unknown fields. A file written by a newer version may hold fields this version does not know, and ignoring them is the safe behaviour. Rejecting them would stop the app for a harmless reason.

**Checkpoint**
- Each invalid file produces exactly the warning from the design table, and the app starts with the default.
- A file with an unknown field starts without warning about it, or with a note that it was ignored. Decide which, and write it in your design.
- The file on disk is not overwritten by the validator. The corrected values are used in memory only, until the user saves.

---

## Exercise 6 — Show the server's context size

**Goal:** the app knows the context size the server was started with, and tells the user if the app's setting is larger.

**Background:** the context size is set when `llama-server` starts. The app cannot always read it from the server, because the exact call depends on the server build. This exercise uses the simplest approach that is honest: the user enters what they started the server with, the app stores it, and the app warns when the two values differ.

**Do**

1. Add a field `serverContextSize` to the config, separate from `contextSize`. Default it to the same value as `contextSize`.
2. In the validator, add a rule: if `contextSize` is greater than `serverContextSize`, add a warning that says the app is configured for more tokens than the server accepts, and suggest the fix in one line.
3. Print the two values at startup, on one line, for example: `context: app 8192, server 8192`.
4. Test the rule by setting `contextSize` to 16384 and `serverContextSize` to 8192. The warning must appear, and the app must still start.

**Observe**
- Why is it not enough to check only `contextSize`? What would a user see if the server had a smaller context than the app's setting, without this warning?
- Which of the two values should the compaction step (Module 4) use? Write your answer before you read Module 4.

**Checkpoint:** the startup line shows both values, and the mismatch warning appears when they differ.

---

## Exercise 7 — Turn a failed connection into a useful message

**Goal:** when a request fails, the user sees which endpoint failed, why, and what to do.

**Do**

1. In `LocalLlamaProvider`, catch the connection-level errors, and turn each into a message with three parts: the endpoint, the kind of failure, and one line of advice.
2. Map the kinds of failure to advice:
   - connection refused → the server is not running, and the port may be wrong,
   - timeout → the server is busy or starting, or the answer is very long; the setting `timeoutSeconds` controls this,
   - not found (404) → the endpoint path is wrong, usually a missing `/v1`,
   - anything else → show the message, and suggest checking the server log.
3. Make the provider rethrow a typed exception with this message, so the loop can show it without a stack trace.

Start from these shells:

```csharp
// The message is for the user: the endpoint, the kind of failure, and one line of advice.
// The loop catches this type and prints Message only.
public sealed class ProviderException(string message, Exception inner) : Exception(message, inner);

// In LocalLlamaProvider: the one place that turns a low-level error into a ProviderException.
private ProviderException Describe(Exception error)
{
    // A refused connection, a timeout, a 404, and anything else: map each to the advice in step 2.
    throw new NotImplementedException(); // students write this
}
```

A trap: C# does not allow `yield return` inside a `try` block that has a `catch`. If `StreamAsync` is an async iterator, move to the next update inside the `try`, and `yield return` it after the `try`.

**Checkpoint:** stop the server, ask a question, and read the message. It names the port, says "connection refused", and suggests starting the server. Start the server again and the question works.

---

## Exercise 8 — Test the whole lesson

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Normal start | valid `settings.json` | no warnings, app answers |
| Missing file | delete `settings.json` | defaults used, file created on first save |
| Missing brace | remove a closing brace | a clear error on load, defaults used, file not overwritten |
| Port as text | `"chatEndpoint": "localhost:8080"` (no scheme) | warning, default used |
| Context too large for server | `contextSize` above `serverContextSize` | warning, app starts |
| Unknown field | add `"futureSetting": 1` | no crash, value ignored |
| Server stopped | stop `llama-server`, ask a question | the failure message from Exercise 7 |
| Path with spaces | notes folder `C:\Users\Test User\notes` | path read correctly |
| Two copies left | search for the port number | only the config default and the server's own script remain |

**Checkpoint:** every row has a written result, and every failure produces a message the user can act on.

---

## Observe (whole lesson)

1. Which value was the hardest to classify as "config" or "code"? What rule would have made the decision easier?
2. What is the cost of a timeout that is too short? What is the cost of one that is too long? Who pays each cost?
3. If you moved the chat model to a different machine, what would have to change in the config, and what would not?

---

## Discuss

- The loop no longer knows the server's address. Is that a benefit or a risk? Who is responsible for the risk when the address is wrong?
- Why does the validator replace bad values instead of stopping the app? When would stopping be the right answer?
- A student suggests storing the API key in the config file "for convenience". Write a two-sentence reply.

---

## Stretch

- Add a `--config <path>` option, so a user can keep several configs. Which field, if any, should be allowed to differ between them?
- Add a `/config` command that prints the active values and their sources (file, default, or fallback). Why is the source useful to a user?
- Write a small script that checks every value in the design table against a sample file. Run it in your CI, if you have one.

---

## Time check

If you are past the planned time, finish Exercises 4 and 5. A config that loads but does not validate is worse than a hardcoded value, because it looks trustworthy.
