# Module 1, Lesson 1 — Review Checklist

Use this list before you mark the lesson complete. Each item is something you can check, not something you believe.

## Layout

- [ ] The RAG code is in slices: `Helpers/`, `Tools/` and any `Models/` are gone, and each moved file's namespace matches its folder.
- [ ] `/eval` gave the same score after the move as before it.
- [ ] Every logic file I wrote in this lesson is free of `Console`, and every screen file's name says it is one.

## Setup and settings

- [ ] I can state, with one example each, what belongs in setup and what belongs in settings.
- [ ] The wizard never downloads from `/settings`.
- [ ] Re-running `/setup` shows the current value of every question, and Enter keeps it.

## Scanning

- [ ] The scan runs by itself, before the first question, and reports one tick line per probe through its `progress` callback. The scanner itself never calls `Console`.
- [ ] On a first run, the scan gets `null` as its main folder, never the default. The agent's own folders are then not checked, and free space is read for every drive.
- [ ] Each building block (`FindLlamaServer`, `ModelFolders`, `FindModels`, `FreeDiskByDrive`, `FindOllamaFolder`, `CheckMainFolder`) can be called on its own, and at least one caller does that without a full scan.
- [ ] The scan uses only read-only commands and does not need administrator rights.
- [ ] The scan makes no network call, runs no program it finds, and checks only the known model folders from README section 3.4 for model files, never the whole disk.
- [ ] I can say, for each known model folder, which tool an end user would have used to fill it.
- [ ] Model files stored as links show their real size, not 0 GB, and no `mmproj-*` file is listed as a model.
- [ ] An installed Ollama is reported, and none of its files are listed.
- [ ] Every external command runs with a timeout, and a timeout does not stop the wizard.
- [ ] A missing `nvidia-smi` is a normal result, and the wizard continues.
- [ ] The scanner returns `null` for any fact it cannot read. It never invents a value.
- [ ] Every scanned value has a source, and the source is shown to the user.
- [ ] I can explain why `AdapterRAM` cannot be trusted above 4 GB, and what the wizard does about it.

## Verify step

- [ ] Every scanned value can be changed, and the value the user confirms or types is the one saved and used.
- [ ] A capped VRAM value gets a prompt that says it is capped.
- [ ] A value the user corrected is not silently replaced by a later rescan.
- [ ] After the table, the wizard always asks about `llama-server` and models. Enter keeps what was found, a typed path is checked and added, and `none` means not installed.
- [ ] When nothing was found, the question lists every folder the scan looked in.
- [ ] Answering `none` to both questions gives no error, and the install guide keeps both steps.

## Folders

- [ ] The main folder is chosen once, in setup, and holds sessions, logs, the `models` folder, and `recent.json`.
- [ ] The main-folder question shows the free space on the folder's drive. Its default is the earlier scan's main folder if that still exists.
- [ ] Choosing an existing main folder adds the models and `llama-server` in it, without a full scan, and the wizard says what it added.

## Earlier scan

- [ ] A completed setup writes `%LOCALAPPDATA%\LocalAgent\last-scan.json`. A cancelled setup leaves it as it was.
- [ ] With `settings.json` deleted and `last-scan.json` kept, the wizard names the earlier scan, offers its main folder, and uses the values I corrected last time, marked `(you)`.
- [ ] A broken `last-scan.json` gives a warning and a first run, and the file is not deleted.
- [ ] I can explain why the earlier scan is in local AppData, and not next to the executable or in the roaming folder.
- [ ] The working folder is asked at every start, with Enter keeping the last one used.
- [ ] The working folder is never stored in `settings.json`.
- [ ] A drive root, the Windows folder, and the main folder are refused as a working folder, with a reason.
- [ ] A user profile folder gets a warning and continues only after a yes.
- [ ] `/folder` during a session starts a new session in the new folder.

## Suggestions

- [ ] The suggestion function is pure: the same input always gives the same output.
- [ ] Rules are checked in order, and the last rule is a safe default.
- [ ] A lower-bound VRAM value produces a lower class, with a reason that says so.
- [ ] Every suggestion includes the estimate warning.
- [ ] A model file the scan found is offered before a download is offered.

## Config file

- [ ] `settings.json` holds only non-secret values, and I have searched it to confirm.
- [ ] The file has a `version` field, and unknown fields are ignored when reading.
- [ ] An invalid value produces a warning and a default, and the file is not deleted.
- [ ] The file is found next to the executable, not by the current working directory.

## Installation guidance

- [ ] The guide only prints text. It starts no process and writes no file.
- [ ] The printed build matches the detected GPU, and the guide says to choose it from the official release page.
- [ ] The guide skips the steps the scan already satisfied (`llama-server` found, a matching model file found).
- [ ] The guide names the `models` and `llama.cpp` folders inside the main folder, and the next `/setup` finds what was put there.
- [ ] Every path in a printed command is quoted, and tested with a folder that has a space.

## Downloads

- [ ] Consent shows the source, the file name, the size, and the destination.
- [ ] The destination is the `models` folder inside the main folder, created only after a yes.
- [ ] Only the exact word `yes` continues. Enter, `y`, and closed input all cancel.
- [ ] The download writes to a `.part` file, and renames it only after success.
- [ ] A size mismatch deletes the `.part` file and reports the error.
- [ ] An existing file with the target name is never replaced silently.
- [ ] I have a written decision on what happens to a `.part` file left by an interrupted run.

## Wiring and checks

- [ ] A first start with no config runs the wizard, and a second start does not. The working-folder question is asked on both.
- [ ] Cancelling `/setup` halfway leaves the config file unchanged.
- [ ] Both endpoints are checked at the end of setup, and each failure has a one-line reason and a suggested fix.

## Edge cases

- [ ] I ran the edge-case table in Exercise 9 and recorded a result for every row.
- [ ] Any unexpected result has been fixed, and the fix is written down.

## Reflection

- [ ] I wrote answers to the five Observe questions, with cases rather than feelings.
- [ ] I can name three assumptions in my wizard that would break on another person's machine.
- [ ] I can explain why the scan comes before the questions, and why the main folder and the working folder are different.
