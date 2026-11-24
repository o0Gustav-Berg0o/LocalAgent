# Module 1, Lesson 1 — Exercises: First-Run Setup

Work through the exercises in order. Each one ends with a **Checkpoint**: a result you must see before you continue. If the checkpoint fails, fix it before moving on, because later exercises depend on it.

**Before you start**
- Run `git status` in the app's repository. Start from a clean state, or make a branch, so you can return to the old version.
- Keep both `llama-server` windows open for Exercises 0 and 8. The rest of the lesson does not need them running.
- Use PowerShell for every command in this lesson, unless a step says otherwise.

**How the lab is laid out**
- Exercises 1 to 3 are about **knowing**: what to ask, what to read, and what to suggest.
- Exercises 4 to 6 are about **storing and acting**: the config file, the install guidance, and the guarded download.
- Exercises 7 to 9 are about **wiring and testing**: startup, `/setup`, and the edge cases.

---

## Exercise 0 — Establish the baseline

**Goal:** know the current state of the app, so you can tell later what the setup changed.

**Why this matters:** in the RAG course, the endpoints and model names live in `Program.cs`. Before you move them into a config file, you need to know exactly what the app does now.

**Do**

1. Start both `llama-server` processes as in the RAG course, Lesson 0. Write down the ports and the model file each one loads.
2. Start the app with `dotnet run`. Write down the first line it prints (the number of notes indexed).
3. Ask one notes question, for example "Who is Naomi Nagata?". Write down the first `[tool]` line in the trace.
4. Open `Program.cs` and find every place that contains one of these values, and write down the line number:
   - the chat endpoint (`http://localhost:8080/v1`)
   - the embedding endpoint (`http://localhost:8081/v1`)
   - the chat model name
   - the embedding model name
   - the notes folder path

**Observe**
- How many places hold each value? If a value appears in more than one place, it is a candidate for the config file, and a candidate for a bug.

**Checkpoint:** you have a table with five values and their line numbers. Keep the table; you will refer to it in Exercise 4.

```text
Value                 Where it is now (file:line)     Current value
chat endpoint         Program.cs:__                   http://localhost:8080/v1
embedding endpoint    Program.cs:__                   http://localhost:8081/v1
chat model            Program.cs:__                   ...
embedding model       Program.cs:__                   ...
notes folder          Program.cs:__ and CommandHelper __
```

---

## Exercise 1 — Design the questions

**Goal:** decide what the wizard asks, why it asks it, and what it does with each answer. No code yet. This exercise is writing, and it is the most important one in the lesson. A wizard with the wrong questions is useless, however good the code.

**Background:** a question is only worth asking if its answer changes what the app does. "What is your name?" changes nothing, so it does not belong in setup. "Do you have an NVIDIA graphics card?" changes which `llama-server` build to suggest, so it may belong, but the scan can often answer it without asking.

**Do**

1. Create a file `setup-design.md` in your app's folder (not in the repository's `samples` folder).
2. Write a table with the following columns: **Question**, **Why we ask it**, **Can the machine answer it instead?**, **Default if the user presses Enter**, **Where the answer is stored**.
3. Write at least these five questions, and add any others you think are needed:
   - What will you mainly use the agent for? (coding, notes, writing, mixed)
   - Where is your project or notes folder?
   - How patient are you with slow answers? (fast, balanced, thorough)
   - Should the wizard download a model for you, or only show you what to download?
   - Do you want to see the hardware scan result before the suggestion? (yes or no)
4. For each question, write one sentence on what the app does differently depending on the answer. If you cannot write that sentence, remove the question.
5. Add a section called **Questions we will not ask**. List at least three things a user might expect to be asked, and say why they are not needed. Examples: name, email, API key, telemetry consent.

**Observe**
- Which question did you almost keep and then remove? What made you change your mind?
- Which answers can the machine determine, so the user does not have to type them?

**Checkpoint:** `setup-design.md` exists, has at least five questions with the five columns filled in, and has the "not asked" section. Show it to a classmate and ask them to explain each question in their own words. If they cannot, rewrite that question.

**Example of a good entry (for orientation, not to copy):**

```text
Question: Where is your project or notes folder?
Why we ask: the indexer and /reindex read files from this folder.
Machine can answer: no, we cannot guess which folder is the user's.
Default: the folder the app was started from, shown in the prompt.
Stored in: settings.json, key "notesFolder".
```

---

## Exercise 2 — Scan the machine (read-only)

**Goal:** collect RAM, GPU, VRAM and free disk space, using commands that only read. Then turn the raw output into a small typed object the rest of the wizard can use.

### Part A: run the commands by hand first

Before writing any C#, see what the system reports. Run each command in PowerShell and write down what you see.

```powershell
# Total physical RAM, in bytes. Divide by 1GB to read it.
(Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory

# Graphics adapters: name, driver version, and the AdapterRAM field.
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion, AdapterRAM

# NVIDIA only: name, total memory in MiB, and driver version.
# If the command is not found, that is a normal result on non-NVIDIA machines.
nvidia-smi --query-gpu=name,memory.total,driver_version --format=csv,noheader,nounits

# Free and used space per drive, in bytes.
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Free, Used
```

**Observe**
- Compare the `AdapterRAM` value with the real VRAM on your card (check the vendor's specification page or the `nvidia-smi` output). If they differ, write down by how much.
- Which of the four commands did you need? Which can fail on some machines?

**Checkpoint:** you have written down the four outputs, and you can explain why the `AdapterRAM` value can be wrong (section 5.1 of the README).

### Part B: define the typed object

Create `Setup/MachineInfo.cs`. The object holds the facts, not the decisions. Decisions come in Exercise 3.

```csharp
namespace LocalAgent.Setup;

// Facts about the machine. Every field can be unknown, so most are nullable.
// "Unknown" is a valid answer and must not stop the wizard.
public record MachineInfo(
    double? TotalRamGb,        // null if the command failed
    string? GpuName,           // null if no GPU was found
    double? VramGb,            // null if unknown; may be a lower bound (see below)
    bool VramIsLowerBound,     // true when the value comes from AdapterRAM and may be capped at 4 GB
    double? FreeDiskGb,        // free space on the drive that holds the app folder
    bool HasNvidiaSmi          // true if nvidia-smi ran successfully
);
```

*Why nullable fields:* a machine can be missing any of these facts. A field that is not nullable forces you to invent a value, and an invented value is worse than "unknown", because the suggestion table will treat it as true.

### Part C: write the scanner with fallbacks

Create `Setup/MachineScanner.cs`. Each fact has a primary source and a fallback. Every source is wrapped in its own `try`, so one failure does not hide the others.

The structure to follow:

```csharp
using System.Diagnostics;
using System.Text.RegularExpressions;

namespace LocalAgent.Setup;

public static class MachineScanner
{
    public static MachineInfo Scan()
    {
        // Each helper returns null on failure. None of them throws to the caller.
        var ramGb = TryReadRamGb();
        var (gpuName, vramGb, lowerBound, hasSmi) = TryReadGpu();
        var freeDiskGb = TryReadFreeDiskGb(AppContext.BaseDirectory);

        return new MachineInfo(ramGb, gpuName, vramGb, lowerBound, freeDiskGb, hasSmi);
    }

    private static double? TryReadRamGb()
    {
        try
        {
            // Write this using Get-CimInstance through a PowerShell process, or through
            // the System.Management namespace if your project references it.
            // Return bytes / 1GB, or null if the value is missing or not a number.
            throw new NotImplementedException(); // students write this
        }
        catch
        {
            return null; // a missing fact is not an error for the user
        }
    }

    // Write TryReadGpu and TryReadFreeDiskGb the same way.
}
```

**Two rules to follow when you fill in the helpers:**

1. **Run external commands with a timeout.** A command that hangs must not hang the wizard. Start the process, wait a fixed number of seconds (for example 5), and kill it if it has not finished. The `Process` class and `WaitForExit(milliseconds)` do this.
2. **Parse, do not trust.** `nvidia-smi` prints `[Not Supported]` for some fields on some drivers. Check each value before you convert it to a number.

**Checkpoint:** run `MachineScanner.Scan()` from a temporary console entry point (or a unit test). It must return a `MachineInfo` on your machine, with a value for every fact you can read, and `null` for any you cannot. Write the output down, and compare it with the Part A outputs.

### Part D: test the failure paths

Pretend the commands fail, and check that the scanner still returns. The simplest way: temporarily rename the executable name you call (for example, call `nvidia-smi-missing`), run, and then change it back.

**Checkpoint:** with `nvidia-smi` unavailable, `HasNvidiaSmi` is `false`, `VramGb` comes from the fallback or is `null`, and the other facts are unchanged.

---

## Exercise 3 — Turn facts into a suggestion

**Goal:** a pure function that takes `MachineInfo` and returns a suggestion and a reason. "Pure" means the same input always gives the same output, and it does not read the disk, the network, or the console. A pure function is easy to test, and that is why it is separated from the scanner.

**Do**

1. Create `Setup/ModelSuggestion.cs` with a record that holds the result:

```csharp
namespace LocalAgent.Setup;

public record ModelSuggestion(
    string HardwareClass,   // a label such as "gpu-6to8gb"
    string ModelKind,       // what kind of model, for example "about 4B"
    string ContextSize,     // for example "8192 tokens"
    string Reason,          // one sentence, shown to the user
    bool IsEstimate         // always true in this course; the estimate is never presented as exact
);
```

2. Write a function `Suggest(MachineInfo info)` that applies the rule table from the README (section 3.3). Start with the lowest class and check upward, so an unknown value falls to the safe default.

3. Write the rules as an ordered list, not a chain of ifs you cannot read. For example:

```csharp
// Rules are checked in order. The first match wins.
// Each rule has a label, a test, and a suggestion.
var rules = new (string Label, Func<MachineInfo, bool> Matches, string Kind, string Context)[]
{
    ("gpu-24gb-plus", i => i.VramGb >= 24 && !i.VramIsLowerBound, "14B or larger", "16384 tokens"),
    ("gpu-12to16gb",  i => i.VramGb >= 12,                        "about 8B",      "8192 tokens"),
    ("gpu-6to8gb",    i => i.VramGb >= 6,                         "about 4B",      "8192 tokens"),
    // ... continue down to the CPU-only class
    ("cpu-only",      i => true,                                  "1B to 3B",      "4096 tokens"),
};
```

Read this carefully. The last rule matches everything, so it is the default. Every rule above it is a claim the code must justify.

4. Notice the detail in the first rule: `!i.VramIsLowerBound`. A 24 GB card reported as 4 GB would otherwise be suggested a 4B model. Decide, and write in a comment, what the wizard should do with a lower-bound VRAM value. (Hint: suggest a class one step lower, and say so in the reason.)

5. Make the reason honest. Every suggestion must include the sentence "This is an estimate. Test the model on your machine." The lab requires it, because the estimates are rough (README section 3.3).

**Checkpoint:** write a test or a temporary console check with these inputs, and confirm the outputs:

| Input | Expected class |
|---|---|
| `VramGb = 16`, not lower bound | `gpu-12to16gb` |
| `VramGb = 4`, lower bound `true` | a class one step below the fact, with a reason saying so |
| `VramGb = null`, `TotalRamGb = 8` | `cpu-only` |
| all fields `null` | `cpu-only`, with a reason saying the machine could not be read |

**Observe**
- Change one rule's boundary (for example, make 6 GB into 8 GB). Which case changed? Did you expect that case to change?
- Why is the table in the README not the same as the code? What should happen when they disagree?

---

## Exercise 4 — Store the answers in a config file

**Goal:** write the answers from Exercises 1 to 3 to `settings.json`, and read them back. Replace the hardcoded values from Exercise 0 with values from the file.

**Do**

1. Create `Settings/AppConfig.cs` with a record that matches the file format in section 3.5 of the README. Include a `Version` field.

```csharp
namespace LocalAgent.Settings;

public record AppConfig(
    int Version,
    string Purpose,
    string ChatEndpoint,
    string ChatModel,
    string EmbeddingEndpoint,
    string EmbeddingModel,
    string NotesFolder,
    int ContextSize,
    bool SetupCompleted
)
{
    // Defaults used when the file is missing or a value is invalid.
    // These are the values from your Exercise 0 table.
    public static AppConfig Defaults => new(
        Version: 1,
        Purpose: "mixed",
        ChatEndpoint: "http://localhost:8080/v1",
        ChatModel: "qwen3-vl-4b",
        EmbeddingEndpoint: "http://localhost:8081/v1",
        EmbeddingModel: "qwen3-embedding-0.6b",
        NotesFolder: AppContext.BaseDirectory,
        ContextSize: 8192,
        SetupCompleted: false);
}
```

2. Write `ConfigStore` with two methods: `Load()` and `Save(AppConfig)`. `Load` returns the defaults if the file does not exist. `Save` writes indented JSON.

3. Write the validation rules in one place. Each rule either accepts a value or replaces it with the default and records a warning:

```csharp
// Rules in one place, so the same checks apply to every read.
// A failed rule does not stop the app. It falls back and records why.
var warnings = new List<string>();
if (!Uri.TryCreate(config.ChatEndpoint, UriKind.Absolute, out _))
{
    warnings.Add("chatEndpoint is not a valid URL; using the default.");
    config = config with { ChatEndpoint = AppConfig.Defaults.ChatEndpoint };
}
// ... one block per field
```

4. Decide where `settings.json` lives. Use the folder of the executable (`AppContext.BaseDirectory`), not the current working directory, because the working directory changes when you start the app from another folder.

5. Test the round trip: save, close the app, open `settings.json` in an editor, change `contextSize` to `4096` by hand, start the app, and check the value it reports.

**Checkpoint:**
- The app reads its endpoints and model names from `settings.json`, not from `Program.cs`.
- Deleting `settings.json` brings the app back up with the defaults.
- A file with `"chatEndpoint": "not a url"` produces a warning and uses the default.
- The file contains no secret values. Search it for the word `key` and confirm there is nothing secret.

**Observe**
- Which of your Exercise 0 values is missing from the file? Why? (Hint: the notes folder may be needed in the file, but the chat model's context length may be a model setting, not an app setting.)

---

## Exercise 5 — Show install steps (print only)

**Goal:** tell the user how to install `llama-server` and where to get a model, without running any installer. This exercise teaches the difference between guidance and action.

**Do**

1. Create `Setup/InstallGuide.cs` with a method `Build(MachineInfo info, ModelSuggestion suggestion)` that returns the text to print. It must not start a process and must not write a file.

2. The text has four parts, in this order:
   - **What you need:** `llama-server` (matching your hardware) and one GGUF model file.
   - **Which build:** based on `GpuName`. An NVIDIA card points to the CUDA build, no GPU points to the CPU build, and any other GPU points to the Vulkan build. Say that the user must pick the matching build from the official release page.
   - **Which model:** the kind from the suggestion, with the estimate warning.
   - **Where to put files:** a suggested folder, quoted, shown as a PowerShell command the user can copy, with the path in quotes.

3. Write each command as a separate line that can be copied on its own. Do not join commands with `&&` in the printed text, because a user on an older PowerShell may not have it.

4. Add a line at the end: "Run `/setup` again after you have installed these, and the wizard will check the endpoints."

**Checkpoint:** run the wizard's guide step with your Exercise 2 output. The text must name the right build for your GPU, mention the estimate, and quote any path that contains a space. Copy one printed command into PowerShell and confirm it runs.

**Observe**
- The guide says "pick the matching build from the release page", and does not give a download link. Why? (Hint: links go stale, and the user should see the page they download from.)
- What would have happened if the guide had run the download itself? Which part of Lesson 1's section 3.7 covers this?

---

## Exercise 6 — The guarded download

**Goal:** download a file only after an explicit yes, to a temporary name, with a size check. This is the most important safety exercise in the lesson. Read section 3.6 of the README before you start.

**Do**

1. Create `Setup/GuardedDownload.cs`. The class takes four things: the source URL, the file name, the expected size in bytes (or `null` if unknown), and the destination folder.

2. Write the consent step as a separate method that returns a bool:

```csharp
// The consent step shows four things and waits for an exact answer.
// It returns true only for the exact text "yes". Enter, "y", or "Yes " do not count.
public static bool AskConsent(Uri source, string fileName, long? sizeBytes, string destinationFolder)
{
    Console.WriteLine($"Source:      {source.Host}");
    Console.WriteLine($"Full URL:    {source}");
    Console.WriteLine($"File:        {fileName}");
    Console.WriteLine($"Size:        {(sizeBytes is null ? "unknown" : $"{sizeBytes / 1_000_000_000.0:F1} GB")}");
    Console.WriteLine($"Destination: {destinationFolder}");
    Console.Write("Type 'yes' to download, anything else to cancel: ");
    return Console.ReadLine() == "yes";
}
```

Read this code and ask: what happens if the user presses Enter by mistake? (Answer: `ReadLine` returns an empty string, which is not `yes`, so the download is cancelled.) Also ask: what happens if `ReadLine` returns `null`, as it does when the input is closed? Your code must handle that too.

3. Write the download to a **temporary name**, in the same folder: `fileName + ".part"`. Only rename it to the real name when the download has finished without an error.

```csharp
var partPath = Path.Combine(destinationFolder, fileName + ".part");
var finalPath = Path.Combine(destinationFolder, fileName);

// Download to partPath with HttpClient, streaming to a FileStream.
// On any exception: delete partPath, rethrow or report, and do NOT create finalPath.
// On success: check the size, then File.Move(partPath, finalPath, overwrite: false).
```

Use `overwrite: false`. If the final file exists, stop and ask the user, rather than replacing it silently. The reason is in section 5.9 of the README.

4. Check the size. If `sizeBytes` is known, compare it with the number of bytes written. A mismatch deletes the `.part` file and reports "download incomplete".

5. Show progress. Print one line that updates every 5 percent, or every 100 MB. Do not print every chunk; that is thousands of lines.

6. Add a checksum step **only if** the publisher gives a SHA-256 value. Compute the hash of the `.part` file, compare it, and stop on a mismatch. Write a comment that says what to do when no checksum is published. (Hint: say so to the user, and let them decide.)

**Checkpoint:** use a small file for the test (a few megabytes from a source you trust, or a local file served by a simple HTTP server you start yourself).
- Answering `no` leaves no file in the destination folder, not even a `.part`.
- Answering `yes` and then pressing Ctrl+C in the middle leaves no file under the real name. (A `.part` file is acceptable only if the next run cleans it up. Decide and write which, in a comment.)
- A size mismatch is reported, and the `.part` file is removed.
- Pressing Enter at the consent prompt cancels.

**Observe**
- Why is the temporary name a better protection than checking the size alone? Give a case where only the size check would fail to catch a broken file.
- Where should a download be allowed to write? Should `C:\Windows` ever be a valid destination? Who decides?

---

## Exercise 7 — Wire it together: startup and `/setup`

**Goal:** the app checks the config on every start. If setup is not complete, it runs the wizard. The user can run the wizard again with `/setup`, and the wizard never overwrites a value without asking.

**Do**

1. In `Program.cs`, load the config as the first thing, before any client is created:

```csharp
var config = ConfigStore.Load();
if (!config.SetupCompleted)
{
    config = await SetupWizard.RunAsync(config);   // you write SetupWizard in this exercise
    ConfigStore.Save(config);
}
```

Read this carefully. `RunAsync` takes the current config, not defaults, so a re-run starts from the user's existing answers.

2. Replace the hardcoded endpoints and model names from Exercise 0 with values from `config`.

3. Add a `/setup` case to `CommandHelper`. It runs the wizard with the current config, and shows each existing value as the default, so the user can press Enter to keep it.

4. In the wizard, every question shows the current value in brackets: `Notes folder [C:\Repos\notes]:`. Enter keeps it. Typing a new value replaces it. Decide and write what happens if the user types a folder that does not exist (hint: ask whether to create it).

5. After the wizard, write the config only once, at the end, with all answers. If the user cancels in the middle, nothing is written, and the old config stays.

**Checkpoint:**
- First start with no `settings.json`: the wizard runs, and the file is created with `SetupCompleted = true`.
- Second start: the wizard does not run.
- `/setup`: every question shows the current value, and Enter keeps it.
- Cancelling halfway through `/setup` leaves `settings.json` exactly as it was. Compare the file's contents before and after.

**Observe**
- What did you have to decide about a re-run that was not decided on a first run? Write each decision down.

---

## Exercise 8 — Verify the endpoints

**Goal:** after setup, check that both servers answer, and report which one does not. A setup that writes a config and never tests it has not finished.

**Do**

1. Write `Setup/EndpointCheck.cs` with a method that sends a GET to each endpoint's models path, for example `{ChatEndpoint}/models`. Use a timeout of 5 seconds. An OpenAI-compatible server such as `llama-server` answers this path. If your build does not answer it, say so in the reason and use a different check you choose, and write which one you used.

2. The method returns one result per endpoint: the name, whether it answered, and a short reason. Do not throw. A connection refused is a normal answer that the user needs to read.

3. Print a small table at the end of the wizard:

```text
Endpoint          Address                       Result
chat              http://localhost:8080/v1      OK
embedding         http://localhost:8081/v1      not reachable (connection refused). Is llama-server running?
```

4. For a failed endpoint, print the most likely fix in one line. A connection refused means the server is not running. A timeout means it is starting or the model is too big. A 404 means the path is wrong, which points to the endpoint string.

**Checkpoint:**
- With both servers running, both rows say OK.
- Stop the embedding server and run the check: its row explains the failure and suggests the fix.

**Observe**
- Why check the endpoints at setup time, when the app will check them again when it starts? What does the setup check give the user that the start-up check does not?

---

## Exercise 9 — Test the edge cases

**Goal:** break the wizard on purpose, in the ways the README warns about, and confirm it survives each one.

Work through this table. For each row, write what happened and whether it was correct.

| Test | How to produce it | Expected result |
|---|---|---|
| `nvidia-smi` missing | rename the call temporarily, or run on a machine without NVIDIA | the wizard continues, `HasNvidiaSmi = false`, guide names the non-CUDA build |
| VRAM reported as about 4 GB | use a card or a fake value of 4 GB | the suggestion is one class lower, with a reason |
| A folder with a space in its path | notes folder `C:\Users\Test User\notes` | every printed command quotes the path |
| A corrupt `settings.json` | write `{ "version": 1, ` (missing brace) | a warning, defaults used, the file is not deleted |
| A newer `settings.json` | add `"version": 2` and an unknown field `"futureSetting": true` | the unknown field is ignored, the known fields are read |
| Empty input at the consent prompt | press Enter | the download is cancelled |
| Closed input at the consent prompt | run the download step with input redirected from an empty file | the download is cancelled, no crash |
| A download interrupted | close the connection halfway (for example with a local server that you stop) | no file under the real name, the `.part` file is removed or reported |
| No write permission in the target folder | choose a folder you cannot write to | a clear error before the download starts, not after |

**Checkpoint:** every row has a written result. Any row that did not behave as expected is a bug. Fix it, then add a sentence to your notes saying what the bug was and what fixed it.

---

## Observe (whole lesson)

Answer these in writing. There are no right answers, but there are weak ones: a weak answer names a feeling ("it felt safer"), a strong answer names a case.

1. Name one place where your wizard asks a question that the machine could have answered. Was it a good question to ask anyway? Why?
2. Name one decision the wizard makes that could be wrong on a real machine. How would the user find out, and how would they fix it?
3. Which step is the most important safety step? Describe what happens if you remove it.
4. Describe the difference between `setupCompleted = false` and a missing config file. Are they the same? Should the app treat them the same?

---

## Discuss

- The wizard reads hardware with commands that can fail, and it must keep going when they do. What other part of an agent should work the same way? (Hint: tools.)
- Why is the model suggestion labelled as an estimate, and what would a user lose if it were presented as a fact?
- A user runs `/setup` and changes the notes folder but keeps the model. Which files change? Which must not? Draw the answer.
- Suppose a second person installs the app on their machine using your wizard. What in your wizard assumes the first person's setup? List at least three assumptions.

---

## Stretch (optional)

- Add a `--non-interactive` mode that reads answers from a file and never asks. How would it handle the download consent step? Should it ever download without a yes?
- Add a scan result cache, valid for one day, and a `/setup --rescan` to force a new scan. What must the cache never contain?
- Write a test that runs the whole wizard with scripted answers and a fake download server. Which parts can you test without a real network?

---

## Time check

If you are past the planned time, stop at the end of the current exercise, write down what is left, and continue next time. Do not skip Exercise 6, the guarded download. It is the one exercise whose failure modes can affect a user's disk.
