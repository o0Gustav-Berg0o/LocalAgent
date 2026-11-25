# Module 1, Lesson 1 — Exercises: First-Run Setup

Work through the exercises in order. Each one ends with a **Checkpoint**: a result you must see before you continue. If the checkpoint fails, fix it before moving on, because later exercises depend on it.

**Before you start**
- **Where the code goes:** `Features/FirstRun/` and `Features/Workspace/`. `AppConfig`, `ConfigStore`, the guarded download, and the `ISlashCommand` interface go in `Shared/`, because later lessons use them. Read README section 3.10, the layout and its rules, before Exercise 2.
- Run `git status` in the app's repository. Start from a clean state, or make a branch, so you can return to the old version.
- Keep both `llama-server` windows open for Exercises 0 and 8. The rest of the lesson does not need them running.
- Use PowerShell for every command in this lesson, unless a step says otherwise.

**How the lab is laid out**
- Exercise 0 records where the app stands, and moves the RAG code into the layout this course uses.
- Exercises 1 to 3 are about **knowing**: what to scan, what to show, what to ask, and what to suggest.
- Exercises 4 to 6 are about **storing and acting**: the config file, the install guidance, and the guarded download.
- Exercises 7 to 9 are about **wiring and testing**: startup, the working folder, `/setup`, and the edge cases.

**Code shells:** each exercise shows the shape of the code you write: the types, the method signatures, and comments that say what each part must do. The bodies throw `NotImplementedException`, so the project builds while you work. You write the bodies. A shell is a guide, not a contract: rename or split it if your design needs that, and write down why. Each shell's namespace, or the comment on its first line, says which folder the file goes in (README section 3.10). The shells leave out `using` lines for the course's own namespaces, such as `LocalAgent.Shared.Config`. Add them, or list them once in a `GlobalUsings.cs` file with `global using`.

---

## Exercise 0 — Establish the baseline, then move the RAG code

**Goal:** know the current state of the app, so you can tell later what the setup changed. Then put the code from the RAG course into the layout this course uses, before you add anything new.

### Part A: record the current values

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
   - the notes folder path (in this course it becomes the working folder, which is chosen at each start and never stored in the file; see Exercises 4 and 7)
5. Run `/eval`, and write down the score. Part B must leave it unchanged.

**Observe**
- How many places hold each value? If a value appears in more than one place, it is a candidate for the config file, and a candidate for a bug.

**Checkpoint:** you have the `/eval` score, and a table with five values and their line numbers. Keep both; Part B uses the score, and Exercise 4 uses the table.

```text
Value                 Where it is now (file:line)     Current value
chat endpoint         Program.cs:__                   http://localhost:8080/v1
embedding endpoint    Program.cs:__                   http://localhost:8081/v1
chat model            Program.cs:__                   ...
embedding model       Program.cs:__                   ...
notes folder          Program.cs:__ and CommandHelper __
```

### Part B: put the RAG code in slices

**Why this matters:** the RAG app has one feature, answering from your notes, so folders by role (`Tools/`, `Helpers/`, `Models/`) were enough. This course adds about twenty features, organised as slices (README section 3.10). Moving the existing code first means every lesson after this one adds to a tree that already has one shape. This is a move, not a rewrite: the same method as the RAG course, Lesson 10.

**Do**

1. Move the files with this table. Your file names may differ a little from the RAG course's; match them by what they do.

| From (RAG course) | To | Why |
|---|---|---|
| `Agent/AgentLoop.cs` | stays in `Agent/` | the loop wires the slices together |
| `Tools/NoteTools.cs` or `Helpers/NoteTool.cs` | `Features/Notes/` | the notes tools are one feature |
| `Tools/UtilityTools.cs` | `Features/Utility/` | the date and calculator tools |
| `Helpers/CommandHelper.cs` and `Helpers/CommandContext.cs` | `Shared/Commands/` | every slice's commands go through it (Exercise 7) |
| a parsed-command record such as `Models/ParsedCommand.cs` | `Shared/Commands/` | only the command parser uses it |
| settings classes such as `SearchSettings` and `AppSettings`, often in `Models/` | `Shared/Config/` | config records live in one place |
| `EvalQuestion`, if it has a file of its own (often in `Models/`) | `Features/Eval/` | the eval set is a feature of its own |
| `LocalAgent.Core/` and the MCP server from Lesson 11 | stay as they are | they are separate projects already |

2. For a file the table does not list, ask which feature it serves. One feature: that slice. Two or more, or the loop: `Shared/`.
3. Change each moved file's namespace to match its folder, for example `LocalAgent.Features.Notes`. Then fix the `using` lines in `Program.cs` and wherever else the build complains. Some IDEs do this for a whole project; in Visual Studio it is called Sync Namespaces.
   - If the build now cannot find a type from `LocalAgent.Core`, such as `Indexer`, the moved file used to find it through its old namespace. The core can use the same namespace as the app's old folder, for example `LocalAgent.Helpers`. Add a `using` line for that namespace, and leave the core's namespaces alone in this exercise.
4. Move one folder at a time, and build after each move. Change nothing else: no renames, no fixes. A move that also fixes something hides both. Close the moved files' editor tabs first: saving an old tab can write the file back to its old folder, and the build then finds every type twice.
5. Run `/eval` again.

**Read this carefully.** The RAG commands (`/search`, `/reindex`, `/eval`) and the `record EvalQuestion` at the bottom of `CommandHelper.cs` move with `CommandHelper` for now. Splitting them into the Notes and Eval slices changes code, not only its place, so it is a stretch task at the end of this lab. If `EvalQuestion` has a file of its own, it goes to `Features/Eval/` anyway, and `CommandHelper` in `Shared/` then uses a slice. That is the one place `Shared/` points at a slice, and the stretch task removes it.

**Checkpoint**
- `Helpers/` and `Tools/` are gone, and so is `Models/` if you had one.
- The app builds, and a notes question gives the same first `[tool]` line as in Part A.
- `/eval` gives the same score as in Part A.

---

## Exercise 1 — Design the questions

**Goal:** decide what the wizard asks, why it asks it, and what it does with each answer. No code yet. This exercise is writing, and it is the most important one in the lesson. A wizard with the wrong questions is useless, however good the code.

**Background:** the wizard now scans first and shows the result before it asks anything. So a question is only worth asking if the machine cannot answer it and the answer changes what the app does. "What is your name?" changes nothing, so it does not belong in setup. "Do you have an NVIDIA graphics card?" is answered by the scan, so it does not need a question at all.

**Do**

1. Create a file `setup-design.md` in your app's folder (not in the repository's `samples` folder).
2. Write a table with the following columns: **Question**, **Why we ask it**, **Can the machine answer it instead?**, **Default if the user presses Enter**, **Where the answer is stored**.
3. Write at least these four questions, and add any others you think are needed:
   - What will you mainly use the agent for? (coding, writing, mixed)
   - Where should the agent keep its own files: sessions, logs, and downloaded models? (the main folder)
   - How patient are you with slow answers? (fast, balanced, thorough)
   - Should the wizard download a model for you, or only show you what to download?

   The scan runs before any question, and its result is always shown, so there is no question about showing it. Add one more row for "Which folder should the agent work in?". In the table, put "not stored in settings.json" under **Where the answer is stored**, and write in **Why we ask it** why this question does not belong in setup (README section 3.8).
4. For each question, write one sentence on what the app does differently depending on the answer. If you cannot write that sentence, remove the question.
5. Add a section called **Questions we will not ask**. List at least three things a user might expect to be asked, and say why they are not needed. Examples: name, email, API key, telemetry consent, and the hardware values (the scan reads them, and the user only corrects them).

**Observe**
- Which question did you almost keep and then remove? What made you change your mind?
- Which answers can the machine determine, so the user does not have to type them?
- Why does the scan come before the questions, and not after? (The answer is in README section 3.9.)

**Checkpoint:** `setup-design.md` exists, has at least five rows (four questions and the working-folder row) with the five columns filled in, and has the "not asked" section. Show it to a classmate and ask them to explain each question in their own words. If they cannot, rewrite that question.

**Example of a good entry (for orientation, not to copy):**

```text
Question: Where should the agent keep its own files?
Why we ask: sessions, logs and downloaded models need one known place, and the user should choose it.
Machine can answer: partly. It can show free disk space on each drive, but not which place the user wants.
Default: C:\Users\<you>\LocalAgent, shown with its free space.
Stored in: settings.json, key "mainFolder".
```

---

## Exercise 2 — Scan the machine (read-only, automatic)

**Goal:** collect everything the machine can tell you, using commands that only read. Then turn the raw output into a small typed object, with the source of each value, that the rest of the wizard can show and use.

### Part A: run the commands by hand first

Before writing any C#, see what the system reports. Run each command in PowerShell and write down what you see.

```powershell
# Windows version: caption and build.
Get-CimInstance Win32_OperatingSystem | Select-Object Caption, Version

# CPU name, physical cores, and logical processors.
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors

# Total physical RAM, in bytes. Divide by 1GB to read it.
(Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory

# Graphics adapters: name, driver version, and the AdapterRAM field.
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion, AdapterRAM

# NVIDIA only: name, total memory in MiB, and driver version.
# If the command is not found, that is a normal result on non-NVIDIA machines.
nvidia-smi --query-gpu=name,memory.total,driver_version --format=csv,noheader,nounits

# Free and used space per drive, in bytes.
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Free, Used

# Is llama-server on the PATH? Prints nothing if it is not.
where.exe llama-server

# Is something already listening on the two endpoint ports?
Get-NetTCPConnection -State Listen -LocalPort 8080,8081 -ErrorAction SilentlyContinue | Select-Object LocalPort

# Model files in the main folder's models folder. Replace the path with yours if it differs.
Get-ChildItem -Path "$env:USERPROFILE\LocalAgent\models" -Filter *.gguf -ErrorAction SilentlyContinue | Select-Object Name, Length
```

**Observe**
- Compare the `AdapterRAM` value with the real VRAM on your card (check the vendor's specification page or the `nvidia-smi` output). If they differ, write down by how much.
- Which of the commands can fail on some machines? Which need no administrator rights?
- Why does the `where.exe` check not run `llama-server` to ask its version? (Hint: section 3.4, the rule about programs the scan finds.)

**Checkpoint:** you have written down the outputs of all the commands, and you can explain why the `AdapterRAM` value can be wrong (section 5.1 of the README).

### Part B: define the typed object

Create `Features/FirstRun/MachineInfo.cs`. The object holds the facts, not the decisions. Decisions come in Exercise 3. The `Sources` dictionary holds, for each fact, where it was read. The verify screen in Part E shows it.

```csharp
namespace LocalAgent.Features.FirstRun;

// Facts about the machine. Every field can be unknown, so most are nullable.
// "Unknown" is a valid answer and must not stop the wizard.
public record MachineInfo(
    string? OsName,                              // Win32_OperatingSystem, for example "Windows 11 Pro"
    string? CpuName,                             // Win32_Processor
    int? CpuCores,                               // physical cores
    double? TotalRamGb,                          // null if the command failed
    string? GpuName,                             // null if no GPU was found
    double? VramGb,                              // null if unknown; may be a lower bound (see below)
    bool VramIsLowerBound,                       // true when the value comes from AdapterRAM and may be capped at 4 GB
    double? FreeDiskGb,                          // free space on the drive that holds the main folder
    bool HasNvidiaSmi,                           // true if nvidia-smi ran successfully
    string? LlamaServerPath,                     // null if where.exe found nothing
    IReadOnlyList<int> PortsInUse,               // endpoint ports (8080, 8081) that already have a listener
    IReadOnlyList<string> ModelFiles,            // *.gguf files found in the known folders
    IReadOnlyDictionary<string, string> Sources  // fact name -> where it was read, shown in the verify step
);
```

*Why nullable fields:* a machine can be missing any of these facts. A field that is not nullable forces you to invent a value, and an invented value is worse than "unknown", because the suggestion table will treat it as true.

### Part C: write the scanner with fallbacks

Create `Features/FirstRun/MachineScanner.cs`. Each fact has a primary source and a fallback. Every source is wrapped in its own `try`, so one failure does not hide the others.

The structure to follow:

```csharp
using System.Diagnostics;
using System.Text.RegularExpressions;

namespace LocalAgent.Features.FirstRun;

public static class MachineScanner
{
    public static MachineInfo Scan()
    {
        // Each helper returns null on failure. None of them throws to the caller.
        // Each helper records its source in the dictionary, and prints one tick line
        // when it finishes. The scanner asks no questions. It returns facts only.
        var sources = new Dictionary<string, string>();

        var osName = TryReadOs(sources);
        var ramGb = TryReadRamGb(sources);
        var (gpuName, vramGb, lowerBound, hasSmi) = TryReadGpu(sources);

        // Free disk is measured on the drive of the default main folder (%USERPROFILE%\LocalAgent).
        // If the user picks another main folder in the verify step, measure again.
        var freeDiskGb = TryReadFreeDiskGb(sources);

        // Add the other helpers the same way: TryReadCpu, TryReadLlamaServer,
        // TryReadPortsInUse, TryReadModelFiles. Each returns a value or null.
        // ...

        return new MachineInfo(osName, /* ...every other fact, in the record's order... */ sources);
    }

    private static double? TryReadRamGb(Dictionary<string, string> sources)
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

    // nvidia-smi first. If it is missing or fails, AdapterRAM from Win32_VideoController,
    // with IsLowerBound = true (README section 5.1).
    private static (string? Name, double? VramGb, bool IsLowerBound, bool HasSmi) TryReadGpu(
        Dictionary<string, string> sources)
    {
        throw new NotImplementedException(); // students write this
    }

    // Runs one read-only command with a timeout (rule 1 below). Returns standard output,
    // or null if the program is missing, exits with an error, or runs longer than timeoutMs.
    // On a timeout, kill the process before returning.
    private static string? TryRun(string fileName, string arguments, int timeoutMs = 5000)
    {
        throw new NotImplementedException(); // students write this
    }

    // Write TryReadOs, TryReadFreeDiskGb and the rest the same way, each one using TryRun.
}
```

**Rules to follow when you fill in the helpers:**

1. **Run external commands with a timeout.** A command that hangs must not hang the wizard. Start the process, wait a fixed number of seconds (for example 5), and kill it if it has not finished. The `Process` class and `WaitForExit(milliseconds)` do this.
2. **Parse, do not trust.** `nvidia-smi` prints `[Not Supported]` for some fields on some drivers. Check each value before you convert it to a number.
3. **Print one tick line per probe.** For example `[ok] total RAM 32 GB (Win32_ComputerSystem)` or `[--] llama-server not found`. The user watches the scan, not a blank screen. The scan must finish in under 30 seconds on a normal machine.
4. **Record the source of every value.** Put it in the `Sources` dictionary, for example `"VRAM" = "nvidia-smi"` or `"VRAM" = "AdapterRAM, capped at 4 GB"`. Section 3.9 of the README says why.

**Checkpoint:** run `MachineScanner.Scan()` from a temporary console entry point (or a unit test). It must return a `MachineInfo` on your machine, with a value for every fact you can read, and `null` for any you cannot. Compare the result with the Part A outputs, and check that each value has a source.

### Part D: test the failure paths

Pretend the commands fail, and check that the scanner still returns. The simplest way: temporarily rename the executable name you call (for example, call `nvidia-smi-missing`), run, and then change it back.

**Checkpoint:** with `nvidia-smi` unavailable, `HasNvidiaSmi` is `false`, `VramGb` comes from the fallback or is `null`, and the other facts are unchanged.

### Part E: show the scan and let the user verify it

The verify screen shows the scan as a table, with the source of each value, and lets the user change any row before anything is suggested or saved (README section 3.9). It is two files, following the split in README section 3.10. `Features/FirstRun/ScanRows.cs` is the logic: the rows, and the check of a typed value. `Features/FirstRun/ScanReviewScreen.cs` is the screen: it prints the table and reads the answers. The screen reads from a `TextReader` and writes to a `TextWriter`, not from `Console` directly, so a test can drive it with scripted input.

```csharp
// Features/FirstRun/ScanRows.cs: logic, no Console.
namespace LocalAgent.Features.FirstRun;

// One row of the verify screen. Key is the config name ("ramGb"), Label is what the user sees ("RAM").
public record ScanRow(string Key, string Label, string Value, string Source, bool ChangedByUser);

// The facts after the user's corrections, and the keys of the rows the user changed.
public record ReviewResult(MachineInfo Info, IReadOnlyList<string> ChangedByUser);

public static class ScanRows
{
    // The rows in the order of the table in README section 3.4.
    // An unknown value is shown as "unknown", never as 0 or an empty string.
    public static IReadOnlyList<ScanRow> ToRows(MachineInfo info, IReadOnlyList<string> changedByUser)
    {
        throw new NotImplementedException(); // students write this
    }

    // Checks one typed value for one row. Returns the updated facts, or an error to show.
    // RAM and VRAM: a positive number. A typed VRAM value sets VramIsLowerBound to false.
    // "Model folders": an existing folder. Its *.gguf files (not its subfolders) join ModelFiles.
    public static (MachineInfo Info, string? Error) Apply(MachineInfo info, string key, string typed)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

```csharp
// Features/FirstRun/ScanReviewScreen.cs: the screen.
namespace LocalAgent.Features.FirstRun;

public static class ScanReviewScreen
{
    // 1. If VramIsLowerBound is true, ask for VRAM first, with the capped prompt from section 3.9.
    // 2. Print the table, then ask: "Type a row name to change it, or press Enter to accept:".
    // 3. For a named row, ask "RAM [32 GB]:". Enter keeps the value. A typed value goes through
    //    ScanRows.Apply. On an error, show it and ask again.
    // 4. Print the table again, with changed rows marked "(you)", and repeat from step 2.
    // Returns null if the user types "cancel" or the input is closed. Nothing is saved then.
    public static ReviewResult? Review(MachineInfo info, TextReader input, TextWriter output)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

What the screen can look like (values are illustrative):

```text
Scan result                                      Source
  Windows         Windows 11 Pro 26100           Win32_OperatingSystem
  CPU             AMD Ryzen 7 5800X, 8 cores     Win32_Processor
  RAM             32 GB                          Win32_ComputerSystem
  GPU             NVIDIA GeForce RTX 3070        nvidia-smi
  VRAM            8 GB                           nvidia-smi
  Free disk (C:)  212 GB                         Get-PSDrive
  llama-server    not found                      where.exe
  Model folders   C:\Users\you\LocalAgent\models (no files)
  Ports in use    none                           Get-NetTCPConnection

Type a row name to change it, or press Enter to accept:
```

**Checkpoint:** drive `Review` with a `StringReader`:
- `"\n"` (Enter at once) returns the scan unchanged, and `ChangedByUser` is empty.
- `"ram\n16\n\n"` returns `TotalRamGb = 16`, `ChangedByUser` holds `"ramGb"`, and the second table shows `16 GB (you)`.
- `"ram\nlots\n16\n\n"` refuses `lots` and asks again.
- A `MachineInfo` with `VramIsLowerBound = true` gets the VRAM question first.
- An empty `StringReader` (closed input) returns `null`, with no exception.

---

## Exercise 3 — Turn facts into a suggestion

**Goal:** a pure function that takes `MachineInfo` and returns a suggestion and a reason. "Pure" means the same input always gives the same output, and it does not read the disk, the network, or the console. A pure function is easy to test, and that is why it is separated from the scanner.

**Do**

1. Create `Features/FirstRun/ModelSuggestion.cs` with a record that holds the result:

```csharp
namespace LocalAgent.Features.FirstRun;

public record ModelSuggestion(
    string HardwareClass,   // a label such as "gpu-6to8gb"
    string ModelKind,       // what kind of model, for example "about 4B"
    string ContextSize,     // for example "8192 tokens"
    string Reason,          // one sentence, shown to the user
    bool IsEstimate         // always true in this course; the estimate is never presented as exact
);
```

2. In the same file, write a static class `ModelSuggester` with a function `Suggest(MachineInfo info)` that applies the rule table from the README (section 3.3). Check from the highest class down, and end with a rule that matches everything, so an unknown value falls to the safe default. Start from this shell:

```csharp
public static class ModelSuggester
{
    public const string EstimateWarning = "This is an estimate. Test the model on your machine.";

    // Pure: reads only its argument. No disk, no network, no console.
    public static ModelSuggestion Suggest(MachineInfo info)
    {
        // 1. Decide which VRAM value the rules see when VramIsLowerBound is true (step 4).
        // 2. Walk the rules in order (step 3). The first match wins.
        // 3. Build the reason: the facts that decided the class, then EstimateWarning.
        //    If every fact is null, say that the machine could not be read.
        throw new NotImplementedException(); // students write this
    }
}
```

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

**Goal:** write the verified answers from Exercises 1 to 3 to `settings.json`, and read them back. Replace the hardcoded values from Exercise 0 with values from the file.

**Do**

1. Create `Shared/Config/AppConfig.cs` with a record that matches the file format in section 3.5 of the README. Include a `Version` field. The main folder is in the file. The working folder is not (section 3.8).

```csharp
namespace LocalAgent.Shared.Config;

public record AppConfig(
    int Version,
    string Purpose,
    string Patience,            // fast, balanced, or thorough
    string MainFolder,
    double? RamGb,              // from the scan, or the user's correction
    double? VramGb,             // from the scan, or the user's correction
    string ChatEndpoint,
    string ChatModel,
    string EmbeddingEndpoint,
    string EmbeddingModel,
    int ContextSize,
    bool SetupCompleted,
    string[] UserSetValues      // config keys the user corrected in the verify step, for example "vramGb"
)
{
    // Defaults used when the file is missing or a value is invalid.
    // The endpoints and model names are the values from your Exercise 0 table.
    public static AppConfig Defaults => new(
        Version: 1,
        Purpose: "coding",
        Patience: "balanced",
        MainFolder: Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.UserProfile), "LocalAgent"),
        RamGb: null,
        VramGb: null,
        ChatEndpoint: "http://localhost:8080/v1",
        ChatModel: "qwen3-vl-4b",
        EmbeddingEndpoint: "http://localhost:8081/v1",
        EmbeddingModel: "qwen3-embedding-0.6b",
        ContextSize: 8192,
        SetupCompleted: false,
        UserSetValues: []);
}
```

2. Create `Shared/Config/ConfigStore.cs` with two public methods: `Load()` and `Save(AppConfig)`. `Load` returns the defaults if the file does not exist. `Save` writes indented JSON. Start from this shell:

```csharp
using System.Text.Json;

namespace LocalAgent.Shared.Config;

public static class ConfigStore
{
    // Next to the executable, not in the current directory (step 4).
    public static string FilePath => Path.Combine(AppContext.BaseDirectory, "settings.json");

    // Missing file: the defaults, no warning. A file that cannot be parsed: the defaults and
    // a warning, and the file stays on disk untouched. Unknown fields are ignored.
    public static (AppConfig Config, IReadOnlyList<string> Warnings) Load()
    {
        throw new NotImplementedException(); // students write this
    }

    // Indented JSON with camelCase names, so the file matches README section 3.5.
    // Hint: JsonSerializerOptions with WriteIndented and JsonNamingPolicy.CamelCase.
    public static void Save(AppConfig config)
    {
        throw new NotImplementedException(); // students write this
    }

    // One rule per field (step 3). Returns the fixed config and the warnings.
    private static (AppConfig Config, List<string> Warnings) Validate(AppConfig config)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

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
// ... one block per field, including mainFolder (a path that is not rooted is replaced)
```

4. Decide where `settings.json` lives. Use the folder of the executable (`AppContext.BaseDirectory`), not the current working directory, because the working directory changes when you start the app from another folder.

5. Test the round trip: save, close the app, open `settings.json` in an editor, change `contextSize` to `4096` by hand, start the app, and check the value it reports.

**Checkpoint:**
- The app reads its endpoints and model names from `settings.json`, not from `Program.cs`.
- The working folder is not in the file.
- Deleting `settings.json` brings the app back up with the defaults.
- A file with `"chatEndpoint": "not a url"` produces a warning and uses the default.
- The file contains no secret values. Search it for the word `key` and confirm there is nothing secret.

**Observe**
- Which of your Exercise 0 values is missing from the file? Why? (Hint: the working folder is the answer for the notes folder. It is chosen at each start and never stored. The chat model's context length may be a model setting, not an app setting.)

---

## Exercise 5 — Show install steps (print only)

**Goal:** tell the user how to install `llama-server` and where to get a model, without running any installer. This exercise teaches the difference between guidance and action.

**Do**

1. Create `Features/FirstRun/InstallGuide.cs` with a method `Build(MachineInfo info, ModelSuggestion suggestion, string mainFolder)` that returns the text to print. It must not start a process and must not write a file. Start from this shell:

```csharp
namespace LocalAgent.Features.FirstRun;

public static class InstallGuide
{
    // The text to print. Starts no process and writes no file.
    // Four parts, in the order of step 2. A part the scan already satisfied is left out (step 5).
    public static string Build(MachineInfo info, ModelSuggestion suggestion, string mainFolder)
    {
        throw new NotImplementedException(); // students write this
    }

    // "CUDA", "Vulkan", or "CPU", from GpuName and HasNvidiaSmi.
    public static string BuildKind(MachineInfo info)
    {
        throw new NotImplementedException(); // students write this
    }

    // Every path in a printed command goes through this (README section 5.5).
    public static string Quote(string path) => $"\"{path}\"";
}
```

2. The text has four parts, in this order:
   - **What you need:** `llama-server` (matching your hardware) and one GGUF model file.
   - **Which build:** based on `GpuName`. An NVIDIA card points to the CUDA build, no GPU points to the CPU build, and any other GPU points to the Vulkan build. Say that the user must pick the matching build from the official release page.
   - **Which model:** the kind from the suggestion, with the estimate warning.
   - **Where to put files:** the `models` folder inside the main folder, quoted, shown as a PowerShell command the user can copy.

3. Write each command as a separate line that can be copied on its own. Do not join commands with `&&` in the printed text, because a user on an older PowerShell may not have it.

4. Add a line at the end: "Run `/setup` again after you have installed these, and the wizard will check the endpoints."

5. Skip what the scan already found. If `LlamaServerPath` is set, print "llama-server found at ..." and leave out the build step. If `ModelFiles` holds a file that fits the suggestion, offer to use it and leave out the download. Do not print a download for something the user already has.

**Checkpoint:** run the wizard's guide step with your Exercise 2 output. The text must name the right build for your GPU, mention the estimate, and quote any path that contains a space. Copy one printed command into PowerShell and confirm it runs. Then run it once with a fake `LlamaServerPath` and check that the build step is gone.

**Observe**
- The guide says "pick the matching build from the release page", and does not give a download link. Why? (Hint: links go stale, and the user should see the page they download from.)
- What would have happened if the guide had run the download itself? Which part of Lesson 1's section 3.7 covers this?

---

## Exercise 6 — The guarded download

**Goal:** download a file only after an explicit yes, to a temporary name, with a size check. This is the most important safety exercise in the lesson. Read section 3.6 of the README before you start.

**Do**

1. Create `Shared/Download/GuardedDownload.cs`. The class takes four things: the source URL, the file name, the expected size in bytes (or `null` if unknown), and the destination folder. In this lesson the destination is the `models` folder inside the main folder. Create that folder only after the user has said yes. The file goes in `Shared/Download/`, not in the slice, because the fetch tool in Module 7, Lesson 22 reuses it. Start from this shell. Steps 2 to 6 fill it in.

```csharp
namespace LocalAgent.Shared.Download;

public enum DownloadOutcome { Cancelled, Completed, SizeMismatch, ChecksumMismatch, TargetExists, Failed }

public sealed class GuardedDownload(
    Uri source,
    string fileName,
    long? expectedBytes,            // null if the publisher gives no size
    string destinationFolder,       // the models folder inside the main folder
    string? expectedSha256 = null)  // null if the publisher gives no checksum
{
    // The whole guarded flow. Consent runs first, inside this method, so no caller can skip it.
    // A file under the final name exists only when the result is Completed.
    public Task<DownloadOutcome> RunAsync(HttpClient http, Func<bool> askConsent, CancellationToken cancel)
    {
        // 1. askConsent()                       step 2. false: Cancelled, nothing written
        // 2. Create destinationFolder           only after the yes
        // 3. TargetExists if the final file is already there
        // 4. Stream to fileName + ".part"       steps 3 and 5, with progress
        // 5. Check the size, then the checksum  steps 4 and 6
        // 6. Rename .part to the final name     File.Move(..., overwrite: false)
        throw new NotImplementedException(); // students write this
    }

    // The consent screen from step 2 is its own file. RunAsync takes it as askConsent and
    // calls it first, so the gate stays inside the download.
}
```

2. Write the consent step as a screen file, `Shared/Download/DownloadConsentPrompt.cs`, with one method that returns a bool:

```csharp
namespace LocalAgent.Shared.Download;

public static class DownloadConsentPrompt
{
    // Shows four things and waits for an exact answer.
    // It returns true only for the exact text "yes". Enter, "y", or "Yes " do not count.
    public static bool Ask(Uri source, string fileName, long? sizeBytes, string destinationFolder,
        TextReader input, TextWriter output)
    {
        output.WriteLine($"Source:      {source.Host}");
        output.WriteLine($"Full URL:    {source}");
        output.WriteLine($"File:        {fileName}");
        output.WriteLine($"Size:        {(sizeBytes is null ? "unknown" : $"{sizeBytes / 1_000_000_000.0:F1} GB")}");
        output.WriteLine($"Destination: {destinationFolder}");
        output.Write("Type 'yes' to download, anything else to cancel: ");
        return input.ReadLine() == "yes";
    }
}
```

The caller hands it to the download: `download.RunAsync(http, () => DownloadConsentPrompt.Ask(source, name, size, folder, Console.In, Console.Out), cancel)`.

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

## Exercise 7 — Wire it together: startup, the working folder, and `/setup`

**Goal:** the app checks the config on every start. If setup is not complete, it runs the wizard: scan, verify, questions. After that, and on every later start, it asks which folder to work in. The user can run the wizard again with `/setup`, and the wizard never overwrites a value without asking.

**Do**

1. In `Program.cs`, load the config as the first thing, before any client is created:

```csharp
var (config, warnings) = ConfigStore.Load();
foreach (var warning in warnings)
{
    Console.WriteLine($"[config] {warning}");
}

if (!config.SetupCompleted)
{
    var result = await SetupWizard.RunAsync(config);   // you write SetupWizard in this exercise
    if (result is null)
    {
        Console.WriteLine("Setup cancelled. Nothing was saved.");
        return;
    }
    config = result;
    ConfigStore.Save(config);
    // Console.WriteLine(EndpointCheck.FormatTable(await EndpointCheck.CheckAllAsync(http, config)));   // Exercise 8
}

var workingFolder = WorkingFolderPrompt.Choose(config.MainFolder, Console.In, Console.Out);   // step 6
if (workingFolder is null)
{
    return; // the user cancelled at the folder question
}
```

Read this carefully. `RunAsync` takes the current config, not defaults, so a re-run starts from the user's existing answers. Inside `RunAsync`, the order is the one in README section 4: scan, show, verify, ask, suggest. Start `Features/FirstRun/SetupWizard.cs` from this shell:

```csharp
using LocalAgent.Shared.Config;

namespace LocalAgent.Features.FirstRun;

public static class SetupWizard
{
    // Returns the new config, or null if the user cancels. The caller saves; this method writes
    // nothing except a downloaded model, and that only after its own yes.
    public static Task<AppConfig?> RunAsync(AppConfig current)
    {
        // 1. MachineScanner.Scan()                         Exercise 2, Part C
        // 2. ScanReviewScreen.Review(...)                  Exercise 2, Part E. null: return null
        // 3. Ask purpose, patience, and main folder        current value in brackets (step 4)
        // 4. ModelSuggester.Suggest(...)                   Exercise 3
        // 5. InstallGuide.Build(...)                       Exercise 5
        // 6. Offer the download, GuardedDownload.RunAsync  Exercise 6
        // 7. Return current with the new values, the verified RAM and VRAM,
        //    UserSetValues from the review, and SetupCompleted = true
        throw new NotImplementedException(); // students write this
    }
}
```

2. Replace the hardcoded endpoints and model names from Exercise 0 with values from `config`.

3. Add a `/setup` command (step 7 shows how commands are added). It runs the wizard with the current config, and shows each existing value as the default, so the user can press Enter to keep it. `/setup` scans again, and shows the new scan next to the saved values. A value the user corrected earlier is marked as set by the user, and the wizard asks before it replaces that value.

4. In the wizard, every question shows the current value in brackets: `Main folder [C:\Users\you\LocalAgent]:`. Enter keeps it. Typing a new value replaces it. Decide and write what happens if the user types a folder that does not exist (hint: ask whether to create it).

5. After the wizard, write the config only once, at the end, with all answers. If the user cancels in the middle, nothing is written, and the old config stays.

6. **Ask for the working folder at every start.** After the config is ready, and on every later start, ask `Working folder [<last one>]:`. Enter keeps the default. The default is the first entry of `recent.json` in the main folder, or the folder the app was started from if the list is empty. Check the answer with the rules in section 3.8 of the README: the folder must exist, a drive root, the Windows folder, and the main folder are refused, and a user profile folder gets a warning. Each accepted folder goes to the top of `recent.json`. Keep five. Never store the folder in `settings.json`. Start from these shells:

```csharp
// Features/Workspace/WorkingFolder.cs: logic, no Console.
namespace LocalAgent.Features.Workspace;

public enum FolderVerdict { Accepted, NeedsYes, Refused }

public record FolderCheck(FolderVerdict Verdict, string Reason);

public static class WorkingFolder
{
    // The rules in README section 3.8. Reads the file system, never changes it.
    public static FolderCheck Check(string path, string mainFolder)
    {
        // Resolve with Path.GetFullPath first, then compare case-insensitively.
        // Refused:  the folder does not exist, a drive root, the Windows folder, the main folder.
        // NeedsYes: the user profile folder itself.
        // A folder you cannot write to is hard to detect without writing. It is fine to leave
        // that case to the write tools in Module 3, and to say so in a comment.
        throw new NotImplementedException(); // students write this
    }
}
```

```csharp
// Features/Workspace/WorkingFolderPrompt.cs: the screen.
namespace LocalAgent.Features.Workspace;

public static class WorkingFolderPrompt
{
    // Asks "Working folder [<default>]:" until WorkingFolder.Check accepts an answer (and the user
    // says yes to a NeedsYes folder), then calls RecentFolders.Remember. Returns null if the user
    // cancels or the input is closed.
    public static string? Choose(string mainFolder, TextReader input, TextWriter output)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

```csharp
// Features/Workspace/RecentFolders.cs: logic, no Console.
namespace LocalAgent.Features.Workspace;

public static class RecentFolders
{
    // recent.json in the main folder: a JSON array of full paths, newest first, at most five.
    // A missing or broken file gives an empty list, not an error.
    public static IReadOnlyList<string> Load(string mainFolder)
    {
        throw new NotImplementedException(); // students write this
    }

    // Puts the folder first, removes a duplicate (case-insensitive), and keeps five.
    public static void Remember(string mainFolder, string folder)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

`recent.json` can look like this:

```json
[
  "C:\\Repos\\sandbox",
  "C:\\Repos\\website"
]
```

7. **`/folder` during a session.** It asks the same question again. A change ends the current session and starts a new one in the new folder, because a session belongs to one folder (Module 4, Lesson 13). Every new slash command in this course is a class in its slice (README section 3.10). This lesson adds the interface and the first two commands. `CommandHelper` keeps the commands from the RAG course, and hands every other name to the registered commands:

```csharp
// Shared/Commands/ISlashCommand.cs
namespace LocalAgent.Shared.Commands;

public interface ISlashCommand
{
    string Name { get; }          // without the slash, for example "setup"
    string Description { get; }   // one line, shown in /help

    // arguments is the text after the name, or "".
    Task RunAsync(string arguments, CancellationToken cancellationToken);
}
```

```csharp
// In CommandHelper. The cases from the RAG course stay as they are.
private readonly Dictionary<string, ISlashCommand> _commands = new(StringComparer.OrdinalIgnoreCase);

// Called once per command from Program.cs. A duplicate name is a programming error, so it throws.
public void Register(ISlashCommand command) => _commands.Add(command.Name, command);

// Called from the default branch of the existing switch (or the last else). False: unknown name.
public async Task<bool> TryRunRegisteredAsync(string name, string arguments, CancellationToken cancellationToken)
{
    if (!_commands.TryGetValue(name, out var command)) return false;
    await command.RunAsync(arguments, cancellationToken);
    return true;
}
```

Start the two commands from these shells:

```csharp
// Features/FirstRun/SetupCommand.cs
namespace LocalAgent.Features.FirstRun;

public sealed class SetupCommand(Func<AppConfig> currentConfig, Action<AppConfig> save) : ISlashCommand
{
    public string Name => "setup";
    public string Description => "Scan the machine again, and review the setup answers.";

    // SetupWizard.RunAsync(currentConfig()). Call save only when it returns a config (step 5).
    public Task RunAsync(string arguments, CancellationToken cancellationToken)
        => throw new NotImplementedException(); // students write this
}
```

```csharp
// Features/Workspace/FolderCommand.cs
namespace LocalAgent.Features.Workspace;

public sealed class FolderCommand(string mainFolder, Action<string> startSessionIn) : ISlashCommand
{
    public string Name => "folder";
    public string Description => "Choose another working folder. Starts a new session there.";

    // WorkingFolderPrompt.Choose(...). A different folder: startSessionIn(folder).
    public Task RunAsync(string arguments, CancellationToken cancellationToken)
        => throw new NotImplementedException(); // students write this
}
```

Register both in `Program.cs`, one line each:

```csharp
commands.Register(new SetupCommand(() => config, updated => { config = updated; ConfigStore.Save(config); }));
commands.Register(new FolderCommand(config.MainFolder, folder => { /* end this session, start one in folder */ }));
```

**Checkpoint:**
- First start with no `settings.json`: the wizard runs, scans first, and the file is created with `SetupCompleted = true`. Then the working-folder question is asked.
- Second start: the wizard does not run, but the working-folder question does. Enter keeps the last folder.
- `/setup`: every question shows the current value, and Enter keeps it.
- Cancelling halfway through `/setup` leaves `settings.json` exactly as it was. Compare the file's contents before and after.
- Typing `C:\` at the working-folder question is refused, with a reason in one line, and the question is asked again.
- `recent.json` holds the folder you last chose, and the next start offers it.

**Observe**
- What did you have to decide about a re-run that was not decided on a first run? Write each decision down.
- Describe a case where storing the working folder in `settings.json` would give the user a wrong result on the next start.

---

## Exercise 8 — Verify the endpoints

**Goal:** after setup, check that both servers answer, and report which one does not. A setup that writes a config and never tests it has not finished.

**Do**

1. Write `Features/FirstRun/EndpointCheck.cs` with a method that sends a GET to each endpoint's models path, for example `{ChatEndpoint}/models`. Use a timeout of 5 seconds. An OpenAI-compatible server such as `llama-server` answers this path. If your build does not answer it, say so in the reason and use a different check you choose, and write which one you used. Start from this shell:

```csharp
using LocalAgent.Shared.Config;

namespace LocalAgent.Features.FirstRun;

public record EndpointResult(string Name, string Address, bool Ok, string Reason, string? Fix);

public static class EndpointCheck
{
    // GET {endpoint}/models with a 5-second timeout. Never throws: a refused connection,
    // a timeout, and a 404 each become a result with a reason and a one-line fix (step 4).
    public static Task<EndpointResult> CheckAsync(HttpClient http, string name, string endpoint)
    {
        throw new NotImplementedException(); // students write this
    }

    // Checks the chat and embedding endpoints from the config, one result each.
    public static Task<IReadOnlyList<EndpointResult>> CheckAllAsync(HttpClient http, AppConfig config)
    {
        throw new NotImplementedException(); // students write this
    }

    // The table in step 3, as text. The caller prints it, so this file never calls Console.
    public static string FormatTable(IReadOnlyList<EndpointResult> results)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

2. The method returns one result per endpoint: the name, whether it answered, and a short reason. Do not throw. A connection refused is a normal answer that the user needs to read.

3. Build a small table with `FormatTable`, and print it after setup. Uncomment its line in `Program.cs` from Exercise 7:

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
| VRAM reported as about 4 GB | use a card or a fake value of 4 GB | the prompt says the value is capped, and the suggestion is one class lower until the user types the real value |
| A folder with a space in its path | main folder `C:\Users\Test User\LocalAgent` | every printed command quotes the path |
| A corrupt `settings.json` | write `{ "version": 1, ` (missing brace) | a warning, defaults used, the file is not deleted |
| A newer `settings.json` | add `"version": 2` and an unknown field `"futureSetting": true` | the unknown field is ignored, the known fields are read |
| Empty input at the consent prompt | press Enter | the download is cancelled |
| Closed input at the consent prompt | run the download step with input redirected from an empty file | the download is cancelled, no crash |
| A download interrupted | close the connection halfway (for example with a local server that you stop) | no file under the real name, the `.part` file is removed or reported |
| No write permission in the target folder | choose a folder you cannot write to | a clear error before the download starts, not after |
| `llama-server` already listening on port 8080 | start a server on 8080 before the wizard | the wizard says the server is running, offers to use it, and skips the install step for it |
| A changed value in the verify step | type `16` at the RAM row | the suggestion uses 16, and `settings.json` stores 16 |
| Working folder typed as `C:\` | type `C:\` at the working-folder question | refused with a reason, and the question is asked again |
| Working folder is the main folder | type the main folder's path at the working-folder question | refused with a reason (section 3.8 of the README) |
| The last working folder no longer exists | delete it, then start the app | the question is asked again, with the reason |

**Checkpoint:** every row has a written result. Any row that did not behave as expected is a bug. Fix it, then add a sentence to your notes saying what the bug was and what fixed it.

---

## Observe (whole lesson)

Answer these in writing. There are no right answers, but there are weak ones: a weak answer names a feeling ("it felt safer"), a strong answer names a case.

1. Name one place where your wizard asks a question that the machine could have answered. Was it a good question to ask anyway? Why?
2. Name one decision the wizard makes that could be wrong on a real machine. How would the user find out, and how would they fix it?
3. Which step is the most important safety step? Describe what happens if you remove it.
4. Describe the difference between `setupCompleted = false` and a missing config file. Are they the same? Should the app treat them the same?
5. Name one fact the scan reads that the user is most likely to correct in the verify step. What would the suggestion be if the user had not corrected it?

---

## Discuss

- The wizard reads hardware with commands that can fail, and it must keep going when they do. What other part of an agent should work the same way? (Hint: tools.)
- The scan comes before the questions. What would change if the questions came first? Which questions would a user answer differently?
- Why is the model suggestion labelled as an estimate, and what would a user lose if it were presented as a fact?
- A user runs `/setup` and changes the main folder but keeps the model. Which files change? Which must not? Draw the answer.
- Why are the main folder and the working folder different? Give one problem that would appear for each if they were one folder.
- Suppose a second person installs the app on their machine using your wizard. What in your wizard assumes the first person's setup? List at least three assumptions.

---

## Stretch (optional)

- Add a `--non-interactive` mode that reads answers from a file and never asks. How would it handle the download consent step? Should it ever download without a yes?
- Add a scan result cache, valid for one day, and a `/setup --rescan` to force a new scan. What must the cache never contain? What happens to a value the user corrected?
- Write a test that runs the whole wizard with scripted answers and a fake download server. Which parts can you test without a real network?
- Turn the RAG commands (`/search`, `/reindex`, `/eval`) into `ISlashCommand` classes in `Features/Notes/` and `Features/Eval/`, as in Exercise 7, and remove their cases from `CommandHelper`. Move `EvalQuestion` to `Features/Eval/` at the same time, if it is still in `CommandHelper.cs`. Afterwards nothing in `Shared/` uses a slice. Check `/eval` before and after.

---

## Time check

If you are past the planned time, stop at the end of the current exercise, write down what is left, and continue next time. Do not skip Exercise 6, the guarded download. It is the one exercise whose failure modes can affect a user's disk.
