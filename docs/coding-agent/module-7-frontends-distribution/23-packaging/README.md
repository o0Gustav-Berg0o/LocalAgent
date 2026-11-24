# Module 7, Lesson 23 — Packaging and First Run

**Slides:** (to be added)

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 1, Lesson 1: the first-run setup and the config file.
- Module 2, Lesson 4: the settings menu and `NO_COLOR`.
- Module 7, Lessons 21 and 22: the core and frontend split, and the fetch rules.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain what a packaged release contains, and what it deliberately leaves out.
2. Produce a self-contained, single-file build for Windows, and say what each option does.
3. Make sure the release contains no path, key, or setting that belongs to the developer's machine.
4. Give the first run a clear, short start: the banner, the setup, and the next step.
5. Write an installation guide for a person who has never seen the project, and test it on a clean folder.
6. Build a release checklist, and attach checksums to the files you publish.

---

## 2. Why this lesson exists

The app works on the developer's machine, where the paths are right, the tools are installed, and the models are running. Another person's machine has none of that. A package is the set of files that makes the app run somewhere else, and the first run is where the user meets it. A package that works only where it was built is not finished.

Packaging also makes decisions visible: what is inside, what is not, and what the user must install themselves. The model servers and the models are not in the package, because they are large, they have their own licences, and the setup from Module 1 already guides their installation.

---

## 3. Concepts

### 3.1 What a release contains

A release is a folder or an archive with:

- the application, as one executable file, with the runtime included if the build is self-contained,
- a default configuration file, with no personal values,
- the skills, commands, and hooks examples the user may copy,
- a README with the installation steps, the first run, and the limits,
- a checksum file for each published file.

A release does not contain the model servers, the model files, the user's notes, the user's sessions, or any key.

### 3.2 Self-contained and single-file

A framework-dependent build needs the right .NET runtime already installed. A self-contained build includes the runtime, so the user installs nothing else. A single-file build packs the application into one executable.

For a Windows release, a self-contained single-file build is the usual choice for a learning tool. The command in Exercise 2 has four parts: `-c Release` (the configuration), `-r win-x64` (the runtime identifier for 64-bit Windows), `--self-contained true` (include the runtime), and `-p:PublishSingleFile=true` (one executable). The single-file overview page lists the same settings and the caveats for each .NET version, and it is linked under Sources below.

The size of a self-contained build is large, often tens of megabytes. That is the cost of not requiring a runtime. Say so in the README.

### 3.3 What must not be in the release

Before publishing, search the release for:

- paths from the developer's machine (for example a folder under the user's name, or the repository path),
- keys, tokens, and passwords,
- the sandbox project, and the test notes,
- debug settings that change behaviour (for example a flag that skips approvals for testing).

The hardcoded paths from Module 2, Lesson 3, are the most common leak. A release that still points at the developer's notes folder is broken on every other machine, and it exposes the developer's folder structure.

### 3.4 The first run

The first run is the user's first impression. It should:

1. show a banner with the name, the version, and one line of purpose,
2. check whether setup has been completed, and run it if not (Module 1),
3. say what it will do next, and wait for the user to confirm,
4. end with a clear next step: the command to ask a first question, and how to get help.

A first run that dumps a configuration file or a stack trace is a failed first run. Every error on the first run must name what to do.

### 3.5 Terminal and output

The banner, the colours, and the layout must work in a plain terminal, and with `NO_COLOR` set (Module 2, Lesson 4). A package for a person who uses a different terminal must be tested in that terminal. Keep a plain theme as the fallback, and let the user choose a theme in the settings.

### 3.6 Checksums

A checksum is a fingerprint of a published file (Module 1, Lesson 1). Publishing a checksum with each file lets the user check that the file they downloaded is the file you published. A checksum does not prove the file is safe. It proves that it is unchanged since you computed the checksum.

Code signing is the stronger guarantee, and it needs a certificate. Mention it in the README as a later step, and do not claim the release is signed unless it is.

### 3.7 The installation guide

The guide is written for a person with a clean Windows machine and no knowledge of the project. It lists, in order:

1. what the app needs (Windows, the model servers, a model file),
2. where to put the app folder,
3. how to start the model servers (link to Module 1),
4. how to start the app and complete the first run,
5. how to ask a first question, and how to get help.

Each step should be tested on a clean folder, by someone who did not write the guide.

---

## 4. Gotchas

**4.1 A release built from a dirty working tree.** Uncommitted changes end up in the release. Build from a clean, tagged commit, and record the commit in the release notes.

**4.2 A release with the test sandbox inside.** The sandbox folder and its test files are not part of the product. Exclude them from the publish output, and check the output folder.

**4.3 A version number that is not updated.** A user cannot tell which release they have. Set the version in the project file, show it in the banner, and check it matches the release name.

**4.4 A release that needs a tool the README does not mention.** A missing prerequisite is the most common installation failure. Test on a clean machine, or a clean user account, before you publish.

**4.5 An antivirus that blocks the single file.** Some tools flag single-file executables. Document it in the README, and do not tell users to disable their security software.

**4.6 A first run that asks too many questions.** A first run with twenty questions loses the user. Setup asks what it needs (Module 1), and the rest is set to sensible defaults with a note.

---

## 5. The lab in outline

You will:

1. write the release checklist,
2. produce a self-contained single-file build,
3. search the output for leaks, and fix them,
4. make the first run clear, and test it with `NO_COLOR` set,
5. write the installation guide, and test it on a clean folder,
6. publish checksums for each file.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Self-contained:** a build that includes the runtime, so nothing else needs to be installed.
- **Single-file:** a build packed into one executable.
- **Runtime identifier:** the name of the target platform for a build, such as a Windows 64-bit identifier.
- **Release:** the set of files published for users to install.
- **Checksum:** a fingerprint of a file, used to check that it is unchanged.
- **Code signing:** a certificate-based guarantee of who published a file.

---

## 7. Summary

- A release contains the app, a clean default config, examples, a README, and checksums.
- It does not contain model servers, models, notes, sessions, or keys.
- Build self-contained and single-file for a Windows release, and say what that costs in size.
- Search for leaks before publishing: paths, keys, the sandbox, and debug switches.
- The first run and the installation guide are tested on a clean machine by someone else.

## Sources

- [Single-file deployment overview](https://learn.microsoft.com/en-us/dotnet/core/deploying/single-file/overview): the single-file option, self-contained builds, and the caveats for each version.
- [dotnet publish](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-publish): the command and its options.

Continue to [exercises.md](exercises.md).
