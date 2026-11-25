# Module 1, Lesson 1 — First-Run Setup

**Time:** about 3 hours (60 min reading, 90 min lab, 20 min quiz and reflection). The lab is longer than in the RAG course because you are building something a real user will run on a machine you have not seen before.

**Prerequisites:**
- Module 0 (the RAG course) complete. You should be able to run `/search`, `/eval`, and the agent loop.
- Both `llama-server` processes can start on your machine (the RAG course, Lesson 0).
- You can read PowerShell output. You do not need to be a PowerShell expert.

**Slides:** (to be added)
**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain the difference between **first-run setup** and **settings**, and why a coding tool needs both.
2. Scan a machine automatically with read-only commands, and explain which of the facts matter for running a local model.
3. Show the scan with the source of each value, let the user **verify or change** it, and save what the user confirmed.
4. Turn the verified facts into a **model suggestion** using a small, written rule set, not guesswork.
5. Store the user's answers in a **config file**, separate from secrets, and read it back on the next start.
6. Separate the **main folder** (the agent's own files, chosen once) from the **working folder** (the project, chosen at every start).
7. Guide an installation **without running anything the user did not approve**, including downloads.
8. Explain why the setup wizard can be run again with `/setup`, and what it must not overwrite silently.

---

## 2. Why this lesson exists

In the RAG course, the models were already running when the program started. The endpoints, the model names, and the embedding dimensions were written directly into `Program.cs`. That works for one developer who knows their machine. It does not work for anyone else.

A coding agent that people install on their own machines has three problems that the RAG course never had to solve:

- **The machine is unknown.** A laptop with 8 GB of RAM and no dedicated GPU behaves very differently from a desktop with 24 GB of VRAM. A model that runs well on one can be unusable on the other.
- **The software is not installed.** The user may not have `llama-server`, or may have a build that does not match their GPU (a CPU-only build on a machine that should use CUDA, for example).
- **Downloads are large and trusted.** A model file is several gigabytes. Downloading the wrong file, or the right file from the wrong place, is a real risk. The user must see and approve what is downloaded.

The first-run setup solves all three by looking, showing, asking, suggesting, and confirming, in that order. The machine is read first, so every question that follows can be answered with facts the user has already seen. Nothing is installed or downloaded until the user has seen what it is.

---

## 3. Concepts

### 3.1 Setup versus settings

These two sound alike, and beginners mix them up. They are different:

| | First-run setup | Settings |
|---|---|---|
| When it runs | Once, on first start. Can be re-run with `/setup`. | Any time, with `/settings`. |
| Purpose | Answer "what can this machine run, and what does the user want?" | Change a value the user has already chosen. |
| Who decides | The scan reports, the user verifies, the wizard suggests, the user confirms. | The user changes single values. |
| Can it download things? | Yes, after explicit confirmation. | No. Settings never download. |
| Writes to | The config file, all values at once. | The config file, one value at a time. |

**Why the split matters:** if settings also downloaded models, a user changing a single value (say, the context size) could trigger a 5 GB download without noticing. Keeping download behind the setup wizard, with its own confirmation step, keeps every download visible.

### 3.2 What the hardware facts mean

A model has to fit into memory to run at a usable speed. There are three kinds of memory that matter, and they are not interchangeable.

**System RAM** is the memory the operating system and your programs use. A model that does not fit in VRAM can run from RAM instead, but much more slowly, because the data has to travel over a slower path.

**VRAM** is the memory on the graphics card. Fast models live here. The number that matters is the *total* VRAM, not how much is free right now, because other programs (a browser, a game, the desktop itself) use some of it.

**Unified memory** is the design used in many laptops with Apple chips and some other systems. The CPU and GPU share one pool of memory. On such a machine, RAM and VRAM are the same thing, and the usable amount for a model is smaller than the total.

**Disk space** matters for the model file itself, plus room for the download and the temporary files made while it is installed.

### 3.3 How big a model needs to be

Three numbers decide whether a model fits:

1. **Parameters**, for example 4B means about four billion numbers in the model.
2. **Quantisation**, the number of bits used to store each parameter. Lower bits means a smaller file and less memory, with some loss of quality. A common default for local use is a 4-bit family.
3. **Context size**, how many tokens the model can see at once. A larger context needs more memory for the cache that the model keeps while it works. This cost is separate from the model file.

A rough rule for a 4-bit model: the model file is about half a gigabyte per billion parameters, and the cache adds more on top, which grows with the context size. **These are estimates for planning, not measurements.** Your setup code should say so to the user, and the lab asks you to check a real model on a real machine.

| Hardware class (rough) | Suggested kind of model | Why |
|---|---|---|
| No usable GPU, 8 GB RAM | Small model (around 1–3B), short context | Runs on the CPU, slowly. Good enough to try the app. |
| 6–8 GB VRAM | Around 4B, moderate context | Fits on the GPU with room for the cache. |
| 12–16 GB VRAM | Around 8B, larger context | More reasoning, still comfortable. |
| 24 GB or more VRAM | 14B or larger | Strongest local answers; expect more setup work. |

**Important:** the table is a starting point written in this lesson. Before you ship a version of this wizard, check the current model listings and their real file sizes. Model names and quantisations change faster than course material does.

### 3.4 Read-only scanning

The scan must not change anything on the machine. This is a deliberate rule, and it has three parts:

- Use commands that **read** information (`Get-CimInstance`, `nvidia-smi --query`, `Get-PSDrive`, `where.exe`, `Get-NetTCPConnection`). Do not install drivers, do not run tools with side effects.
- Do not require administrator rights. If a fact needs admin rights to read, say so and skip it.
- Do not send the scan results anywhere. They stay on the machine, in the config file the user can read and delete.

**What the scan covers.** The scan runs by itself, with no questions. It probes the machine one fact at a time, and each probe prints a tick when it finishes.

| Fact | How it is read | What it is used for |
|---|---|---|
| Windows version | `Win32_OperatingSystem` | information, and the install guide's file names |
| CPU name and cores | `Win32_Processor` | information, and the CPU-only fallback |
| Total RAM | `Win32_ComputerSystem` | the model suggestion |
| GPU name and driver | `Win32_VideoController`, `nvidia-smi` | the build named in the install guide |
| VRAM | `nvidia-smi`, or `AdapterRAM` as a lower bound | the model suggestion |
| Free and total disk, per drive | `Get-PSDrive` | checking that a model fits on the main folder's drive |
| `llama-server` | `where.exe llama-server` | skipping an install step that is not needed |
| Model files (`*.gguf`) | the main folder's `models` folder and the app folder | offering a model the user already has |
| Endpoint ports 8080 and 8081 | `Get-NetTCPConnection -State Listen` | reporting a server that already runs |

Four rules for the scan:

- It makes **no network calls**. Nothing leaves the machine, and nothing is looked up online.
- It **does not run** any program it finds. It checks that a file exists and reads its name. The only programs it runs are read-only system queries (`nvidia-smi`, PowerShell queries).
- It **does not search the whole disk** for model files. A full search takes minutes and reads folders the user never meant to share with the agent. A folder the scan should also check is added by the user in the verify step (section 3.9).
- It runs **without administrator rights**. A probe that needs them is skipped and shown as unknown.

### 3.5 The config file

The wizard writes its answers to one file, for example `settings.json`, in the app's folder. The rules for that file:

- **Only non-secret values.** Endpoints, model names, paths, sizes, and choices. No API keys, ever.
- **Human-readable.** Indented JSON, with a comment-free format a user can edit in any text editor.
- **Versioned.** A `"version"` field, so a later release can change the format and still read old files.
- **Validated on read.** A value that is missing or invalid gets a clear message and a default, not a crash.
- **Holds the main folder, not the working folder.** The working folder is chosen at each start (section 3.8), so it never goes in this file. The verified hardware values (`ramGb`, `vramGb`) do go in, because the suggestion depends on them. `userSetValues` lists the ones the user corrected.

An example of what the file might hold (values are illustrative):

```json
{
  "version": 1,
  "purpose": "coding",
  "patience": "balanced",
  "mainFolder": "C:\\Users\\you\\LocalAgent",
  "ramGb": 32,
  "vramGb": 8,
  "chatEndpoint": "http://localhost:8080/v1",
  "chatModel": "qwen3-vl-4b",
  "embeddingEndpoint": "http://localhost:8081/v1",
  "embeddingModel": "qwen3-embedding-0.6b",
  "contextSize": 8192,
  "setupCompleted": true,
  "userSetValues": ["vramGb"]
}
```

### 3.6 Consent for downloads

A download is an action with consequences: disk space, time, and trust in the source. The wizard follows this order, and each step must succeed before the next one starts:

1. **Show the source.** The full web address, and the domain.
2. **Show the file name and size.** Size in gigabytes, rounded up.
3. **Show the destination.** The exact folder on disk. In this lesson the destination is the `models` folder inside the main folder.
4. **Ask for a yes.** Only the exact word or key the wizard asked for counts. Anything else cancels.
5. **Download to a temporary name**, and rename only when the download is complete.
6. **Verify**, if the publisher gives a checksum. Compare it, and stop on a mismatch.

Step 5 matters more than it looks. A download that stops halfway, and is left under the real file name, looks complete to the next program that opens it. The temporary name prevents that.

### 3.7 Guided installation, not automatic installation

The wizard tells the user how to install `llama-server` and where to get a model. For this lesson, the wizard **prints** commands and links. It does not run an installer. Running an installer is a separate decision with its own confirmation, and it is out of scope here. Module 2 (cowork mode) revisits it.

If the scan found `llama-server` or a matching model file, the guide says so and leaves out the step for it.

### 3.8 The main folder and the working folder

Setup asks for one folder. The working folder is chosen at every start. The two are different, and the lesson keeps them apart on purpose.

- **Main folder.** The agent's own space. It holds the files the agent makes for itself: sessions, logs, downloaded models in a `models` subfolder, and a short list of recently used working folders (`recent.json`). The user chooses it once, in setup. The default is `%USERPROFILE%\LocalAgent`.
- **Working folder.** The project the agent reads and edits. This is the project root from Module 2, Lesson 5. The user chooses it at every start. Enter keeps the last one used. It is never stored in `settings.json`, because it changes with each project.

Why keep them apart:

1. **The agent must not read its own files as project files.** If the two were the same, the agent would index its own session logs and its downloaded models.
2. **The agent must not write its state into a project.** Session files written into a git repository make every commit noisy, and can put a conversation into a shared repository.
3. **A project is chosen, not configured.** A user who works on three projects should switch folders at the start, not edit a setting.

The working-folder check runs each time the folder is chosen:

- The folder must exist. If it does not, the wizard says so and asks again.
- A drive root (such as `C:\`), the Windows folder, and the main folder itself are refused, with a reason.
- A user profile folder (for example `C:\Users\you`) gets a warning: the agent could read everything in it. It continues only after a yes.
- A folder the app cannot write to is accepted, with a warning. The read tools still work. The write tools fail later with a clear message, which is better than refusing a folder the user needs to read.

During a session, `/folder` asks the same question. A change ends the current session and starts a new one in the new folder, because a session belongs to one folder (Module 4, Lesson 13).

Changing the main folder moves nothing. Existing sessions and models stay where they are, and the app says so when the user changes it.

### 3.9 Verify before you save

The scan is an estimate of the machine. Some values are wrong (section 5.1), and some are missing. So the scan is never used silently.

- The scan result is shown **before** any question. Setup asks only what the machine cannot answer.
- Each row shows its value and its source, for example `8 GB (nvidia-smi)`. The user can judge the row from that.
- **Enter accepts the scan. Typing a row's name changes that row**, and Enter at the row keeps its old value. A corrected value is the one saved and used for the suggestion, and the table shows it marked `(you)`.
- A capped VRAM value is asked for first, before the rest, with a clear prompt: "VRAM: at least 4 GB (Windows caps this value). Enter keeps 4, or type the value from your GPU's specification."
- The model-folder row accepts more folders, so the user can point the wizard at a place the scan did not check.
- A value the user corrected is listed in `userSetValues` in the config. A later rescan (`/setup --rescan`, a stretch task in the exercises) asks before it replaces that value.
- Nothing is saved until the wizard reaches its end. Cancelling in the middle leaves the config as it was, the same rule as `/setup`.

### 3.10 Where the code goes

From this lesson on, the app is organised by feature, not by kind of code. Each feature is a folder under `Features/`, called a slice, and most lessons add one. A lesson's code sits in one place, so you can read it, review it, and undo it as one piece. Exercise 0 moves the code from the RAG course into this layout first, so the whole app has one shape.

```text
LocalAgent/                 the console app from the RAG course
  Agent/                    the loop from the RAG course. It wires the slices together.
  Features/
    Notes/  Utility/  Eval/ from the RAG course, moved in Exercise 0
    FirstRun/               this lesson: scan, verify screen, suggestion, install guide, wizard
    Workspace/              this lesson: the working folder and the recent folders
    Settings/  ReadFiles/  Permissions/  FileChanges/  Shell/  Context/  ProjectMemory/
    Sessions/  Cowork/  Handoff/  Skills/  CustomCommands/  Hooks/  Subagents/  Mcp/  Fetch/
  Shared/                   code that two or more slices use: Commands/ (CommandHelper), Config/, ...
  Program.cs                builds everything, and registers the slash commands
LocalAgent.Core/            from the RAG course: search, indexer, vector store
```

Five rules:

1. **A slice uses `Shared/`, never another slice.** When a second slice needs a file, the file moves to `Shared/`. When a lesson already knows that a later lesson needs a file, it puts the file in `Shared/` from the start, and says why.
2. **The loop and `Program.cs` may use any slice.** They are where the slices are wired together.
3. **Logic files never call `Console`.** When they must talk to the user, they take a `TextReader` and a `TextWriter`, or they return text for the caller to print.
4. **Screen files draw or read input, and their names say so.** They end in `Screen`, `Prompt`, `Menu`, `Wizard`, or `Command`. A slash command is typed input, so it counts as a screen file.
5. **Every new slash command is a class in its slice** that implements `ISlashCommand`, registered with one line in `Program.cs` (Exercise 7). `CommandHelper` keeps the commands from the RAG course, and hands every other name to the registered commands.

The namespace follows the folder: `Features/FirstRun/MachineScanner.cs` is in `LocalAgent.Features.FirstRun`. In Module 7, Lesson 21, the logic files move into the `LocalAgent.Core` project, into the same folders and with the same namespaces, and the screen files stay in the app. Rule 3 is what keeps that move small. This follows the rule from the RAG course, Lesson 10: code moves to the library when a second frontend needs it, and not before.

---

## 4. The flow you will build

```
first start
  │
  ├─ settings.json exists and setupCompleted = true? ── yes ──► ask for the working folder ──► start
  │
  no
  │
  ▼
scan the machine (automatic, read-only, no questions yet)
  │
  ▼
show the scan: each value with its source, one tick per probe
  │
  ▼
verify or change: Enter keeps a value, typing replaces it
  │
  ▼
ask what the machine cannot answer (purpose, patience, main folder)
  │
  ▼
suggest a model (rule table, with the estimate warning)
  │   a matching model file found by the scan? offer to use it, no download
  ▼
show install steps (print only, skipping what the scan found)
  │
  ▼
offer download? ── no ──► skip, record the choice
  │ yes
  ▼
show source, name, size, destination ── confirm? ── no ──► cancel, record the choice
  │ yes
  ▼
download to temp name, verify, rename
  │
  ▼
write settings.json, setupCompleted = true
  │
  ▼
test both endpoints, report result
  │
  ▼
ask for the working folder (main folder is refused) ──► start the app
```

Every box that changes something has a yes/no before it. Every decision is written to the config file, so a second run does not ask the same setup questions again. The working-folder question is the one exception: it is asked at every start, on purpose (section 3.8).

---

## 5. Gotchas you will meet

These are the mistakes that cost people hours. Read them before the lab.

**5.1 The GPU memory field lies above 4 GB.** `Win32_VideoController.AdapterRAM` is stored in a 32-bit field, so it cannot report more than about 4 GB. A card with 8 GB or 16 GB can show `4294967295` or a value near 4 GB. For NVIDIA cards, prefer `nvidia-smi`. For other cards, report the value as "unknown, at least 4 GB" rather than guessing, and let the user correct it in the verify step.

**5.2 `nvidia-smi` may be missing.** It comes with the NVIDIA driver. On a machine without an NVIDIA card, or with a driver that is not installed, the command does not exist. Your code must treat "command not found" as a normal result, not an error that stops the wizard.

**5.3 A CPU-only build on a GPU machine.** The `llama-server` build you download has to match the hardware: a CUDA build for NVIDIA, a Vulkan or other build for other GPUs, a CPU build otherwise. A CPU build on a machine with a good GPU runs, but slowly, and the user thinks the model is the problem.

**5.4 A model that is too big is worse than a small one.** When a model does not fit in memory, the system starts moving data between RAM and disk, and generation can take minutes per answer. The wizard should prefer a smaller model over a model that barely fits.

**5.5 Paths with spaces and non-ASCII characters.** Folders such as `C:\Users\Some Name\Documents` break commands that are not quoted. The Swedish letters `å`, `ä`, `ö` are also a common source of trouble in older tools. Quote every path you show, and test with a folder that has a space.

**5.6 Antivirus and SmartScreen.** Some security tools block or delay a download of an executable or a large file. A timeout or a missing file after download is often this, not a bug in your code. The wizard should say so in its error message.

**5.7 A partial download.** A connection that drops after 2 GB of a 5 GB file leaves a file that looks real. The temporary-name rule in section 3.6 exists for this reason. Also store the expected size, and compare it after download.

**5.8 A settings file written by a newer version.** If a user runs an older app on a file written by a newer one, unknown fields must be ignored, not rejected. The `version` field tells you what to expect.

**5.9 Re-running setup overwrites real work.** `/setup` must not delete the main folder path or a model choice without asking. Show the current value, and let the user keep it.

**5.10 A scan that reads too much.** A search of the whole disk for `.gguf` files finds every model on the machine, takes minutes, and reads folders the user never meant the agent to see. The scan checks only the main folder's `models` folder and the app folder. A folder beyond those is added by the user, in the verify step.

**5.11 One folder playing two roles.** If the main folder and the working folder are the same, the agent indexes its own sessions and downloaded models as project files, and writes its state into the project. The check in section 3.8 refuses this. Without the check, the bug would only show up later, when session files appear in search results.

---

## 6. Glossary

- **Quantisation:** storing model numbers with fewer bits, to make the file and the memory use smaller.
- **VRAM:** memory on the graphics card.
- **Unified memory:** one memory pool shared by CPU and GPU.
- **Context size:** the number of tokens the model can see at once, including the cache cost.
- **GGUF:** the file format used for local models in `llama.cpp`.
- **Checksum:** a short fingerprint of a file (for example SHA-256). Equal checksums mean the same bytes.
- **Config file:** a file the app reads on start to find its settings.
- **Consent gate:** a yes/no step that must succeed before an action with side effects.
- **Main folder:** the agent's own folder for sessions, logs, and models. Chosen once, in setup.
- **Working folder:** the project the agent reads and edits. Chosen at every start.
- **Verify step:** the screen where the user checks each scanned value and changes any that are wrong.

---

## 7. What you will have at the end

- A `/setup` command that scans the machine by itself, shows the result for the user to verify or change, asks the few questions the machine cannot answer, suggests a model, and shows install steps.
- A working-folder question at every start, and `/folder` during a session. The main folder stays apart from it.
- A download step that only runs after an explicit yes, and only to a temporary file in the `models` folder.
- A config file that stores non-secret answers, the main folder, and the verified hardware values, and is read on every start.
- A start-up check that skips setup when the config says it is complete.
- A short note for the course: what the wizard can and cannot know about a machine.

---

## 8. Summary

- Setup answers "what can this machine run?" once. Settings change single values later. The working folder is chosen at each start and is never a setting.
- The scan runs first, by itself. It is read-only, makes no network call, runs no program it finds, and needs no admin rights.
- The user verifies or changes every scanned value before the suggestion is made.
- Model size is estimated from parameters, quantisation, and context. Estimates are labelled as estimates.
- Downloads need a clear yes after the user sees the source, the name, the size, and the destination.
- Files go to a temporary name first, and are renamed only when complete.
- The config file holds non-secret values, the main folder, and the verified hardware values. It carries a version number.
- The main folder is the agent's own space, set once. The working folder is the project, chosen at each start.

Continue to [exercises.md](exercises.md) for the lab. It is long on purpose: work through it in order, and do not skip the Observe and Discuss sections. They are where the lesson's reasoning is tested.
