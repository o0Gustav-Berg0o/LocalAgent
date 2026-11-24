# Module 7, Lesson 23 — Exercises: Packaging and First Run

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- Start from a clean git state. Commit everything, and tag the commit, so the release has a known source.
- Install the .NET SDK version the project uses. Check the publishing documentation for that version before you use the flags in Exercise 2.
- Have a second Windows user account, or a clean folder on another drive, for the install test in Exercise 5.

---

## Exercise 0 — Write the release checklist

**Goal:** a checklist that is run every time, before any release, so nothing depends on memory.

**Do**

1. Create `RELEASE.md` in the repository. Write the steps in order: check the working tree is clean, update the version, run the tests, build, search for leaks, test the first run, write the installation guide, compute checksums, and tag the commit.
2. For each step, write the command or the check, and what a pass looks like.

**Checkpoint:** the file lists at least eight steps, each with a pass condition.

---

## Exercise 1 — Set the version and the banner

**Goal:** the version is set in one place, and the banner shows it.

**Do**

1. Set the version in the project file.
2. Make the banner read the version from the assembly, not from a second copy of the number.
3. Show the banner with the name, the version, and one line of purpose.

**Checkpoint:** the banner shows the version from the project file, and changing the project file changes the banner after a build.

---

## Exercise 2 — Build a self-contained single file

**Goal:** a release build that runs on a machine without the .NET runtime installed.

**Do**

1. Run the publish command with the Release configuration, a Windows 64-bit runtime identifier, the self-contained option, and the single-file option. Check the publishing documentation for your SDK version to confirm the exact option names before you run it.
2. Write the command you used in `RELEASE.md`, exactly as it was run.
3. Note the size of the output file, and write it in the README as the cost of a self-contained build.

**Read this carefully.** The command must be run from a clean working tree, and the output folder must be inside the repository's publish folder, not in the source folder.

**Checkpoint:** the output is one executable, and it runs on a machine or user account where the .NET runtime is not installed (test this, do not assume it).

---

## Exercise 3 — Search the release for leaks

**Goal:** the release holds no path, key, sandbox, or debug switch that belongs to the developer.

**Do**

1. Search the publish output and the default configuration for your user name, the repository path, the sandbox name, and any key-like string (use the patterns from Lesson 12).
2. Search the source for debug switches that skip approvals or change behaviour, and confirm none are enabled in the Release configuration.
3. Fix each leak. Record each in `RELEASE.md` with what was changed.

**Read this carefully.** The most common leak is a hardcoded path from Module 2, Lesson 3, that was never moved into the config. The search finds it. Fix the code, not only the output.

**Checkpoint**
- The search returns no developer path, no key-like string, and no sandbox reference.
- The default configuration has relative or neutral values, with the notes folder left to the first run.

---

## Exercise 4 — The first run

**Goal:** a first run that a new user can complete, in a plain terminal, with no colour.

**Do**

1. Delete the user's application data folder (on your test account), so the first run is a real first run.
2. Start the release file. Check the banner, the setup questions, the scan, the suggestion, and the guide.
3. Repeat with `NO_COLOR` set, and check that the output is readable and the selection marker is still visible.
4. Note every point where you hesitated, or where the next step was unclear. Fix each one.

**Checkpoint:** a first run completes with no stack trace, and the next step is named at the end.

---

## Exercise 5 — The installation guide and a clean install

**Goal:** a person who did not write the guide can install and start the app.

**Do**

1. Write `INSTALL.md`, following the outline in the README (section 3.7). Include the model servers as a prerequisite, with a link to Module 1, and the folder for the app.
2. Give it to a classmate, or follow it yourself on a clean user account. Do not use anything you remember from development.
3. Write down each step where you had to guess. Fix the guide for each.

**Read this carefully.** A guide tested by its author is not tested. The clean account is the test that matters, and the notes of each guess are its result.

**Checkpoint:** the clean install completes from the guide alone, and the notes of guesses are fixed.

---

## Exercise 6 — Checksums

**Goal:** each published file has a checksum, and the user can check it.

**Do**

1. Compute the SHA-256 of the executable and of the other release files. Write them in a file named `CHECKSUMS.txt`, one line per file.
2. Check the checksum on the clean account, by computing it again and comparing.
3. Write in the README that a checksum shows the file is unchanged, and does not show it is safe.

**Checkpoint:** the checksum on the clean account matches the one in `CHECKSUMS.txt`, and a changed byte in a test copy produces a different value.

---

## Exercise 7 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Runtime not installed | run on the clean account | runs, with no missing-runtime error |
| `NO_COLOR` set | run with the variable set | plain output, marker visible |
| Notes folder with a space | choose a folder with a space in setup | path works in commands and config |
| Security software blocks the file | test on a machine with a strict policy, if available | documented in the README, no advice to disable security software |
| Old configuration from a previous version | a config with an older version number | read with defaults, warning shown |
| Release run from a different folder | run from the desktop | config found next to the executable |
| Version mismatch | a release name that differs from the banner | caught in the release checklist |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. Which step in the release checklist is easiest to skip? What would you add to make it harder to skip?
2. The installation guide needed fixes after the clean install. What does that say about guides written by their authors?
3. The release is larger than you expected. Which part of the size is the runtime, and is that the right trade for your users?

---

## Discuss

- A release is a promise to a user. What does it promise, and which parts of the promise can you check?
- Checksums and signing both talk about trust. What does each prove, and what does neither prove?
- The model servers and the model files are not in the release. Is that the right choice for a learning tool, and who does it exclude?

---

## Stretch

- Add an automatic release script that runs the checklist and stops at the first failed step.
- Add an update check that tells the user when a newer version exists, with the user's consent for each check.
- Build a second release for another platform, and list the differences in `RELEASE.md`.

---

## Time check

If you are past the planned time, finish Exercises 2, 3, 4, and 5. The self-contained build, the leak search, and the clean install are what decide whether anyone else can use the app.
