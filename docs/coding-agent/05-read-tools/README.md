# Module 2, Lesson 5 — Read-Only Tools: Read, List, Search

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 2, Lessons 2 to 4: streaming, the provider seam, and the settings menu.
- Course 1, Lesson 5 (tool design): descriptions and naming.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Build three read-only tools (read a file, list a folder, search text) that a model can call safely.
2. Confine every path to a project root, and explain how a path can escape it.
3. Limit tool output by size and by line count, and tell the model when output was cut.
4. Skip folders and files that should never be read (build output, version control, binaries).
5. Write descriptions that teach the model when to use each tool, and when not to.
6. Measure the token cost of a tool's output, and explain why the cost matters more than the code.

---

## 2. Why this lesson exists

An agent that can only answer from its own training is limited. A coding agent has to look at the code it is asked about. That means reading files, finding the right file, and searching for a name across a project.

Reading is the least dangerous thing an agent can do, but it is not harmless:

- A path that escapes the project lets the model read files the user never meant to share, such as keys or other projects.
- A file that is too large fills the context window in one step, and the model loses everything else it was told.
- A search across a folder of build output returns thousands of matches that say nothing useful.

This lesson gives the model eyes, and draws the boundary of what those eyes can see.

---

## 3. Concepts

### 3.1 The three tools

| Tool | What it does | What the model should use it for |
|---|---|---|
| `read_file` | returns the text of one file, from a start line for a number of lines | opening a file the model has already found |
| `list_files` | returns the names in a folder, optionally filtered by a pattern | discovering what exists before reading |
| `search_text` | returns the lines that contain a given text, with file and line number | finding a name, a call, or an error message across the project |

Each tool does one thing. A tool that reads and searches at once is harder for the model to choose correctly, and harder for you to test.

### 3.2 Paths and the project root

The project root is the folder the user chose to work in. Every path the model gives must resolve to a location inside that root. Paths can escape in several ways:

- `..\..\secrets.txt` climbs above the root.
- An absolute path such as `C:\Users\someone\.ssh\id_rsa` ignores the root entirely.
- A symbolic link or a junction inside the project points somewhere else.
- A path with a different letter case, or a short name (`PROGRA~1`), may slip past a naive check.

The rule: **resolve the full path first, then check that it starts with the root.** Checking the text of the path before resolving it is not enough. The check must happen on the resolved path, and the comparison must be case-insensitive on Windows.

### 3.3 Output limits

Tool output goes into the context, which is limited. Three limits apply:

- **Size in bytes** per call, for example 32 KB.
- **Lines** per call, for example 200 lines for a read.
- **Matches** per search, for example 50 matches.

When a limit is hit, the tool must say so in the output: how much was cut, and how to get the rest. A model that does not know output was cut will reason as if it saw everything, and give confident answers from partial evidence.

### 3.4 What to skip

Some folders contain output that no one wants to read:

- `bin`, `obj`, `node_modules`, `.git`, `.vs`, and similar build and tool folders,
- binary files (images, compiled code, archives), which the model cannot read as text.

Skipping them is a configuration decision, and the list belongs in the config file, so the user can change it. Detecting binary files is a check on the first few bytes of the file, not on the extension alone.

### 3.5 Descriptions teach the model

The model decides which tool to call from the name and the description, and the descriptions are prompts. A good description says:

- what the tool returns,
- when to use it,
- when not to use it, and what to use instead.

"Search" is a poor description. "Find lines containing a text across project files. Use it to locate a name or message before reading a file. Do not use it to read a whole file; use read_file for that." teaches the difference.

Course 1, Lesson 5 showed that descriptions change behaviour. The same applies here, and Exercise 6 measures it.

### 3.6 Token economy

Every character a tool returns costs tokens. A read of a 3000-line file costs more than the answer it supports. The economy rules for this lesson:

- prefer a list and a line range over a whole file,
- search first, read second,
- return the smallest useful unit (a function, not a file),
- keep the output format plain and compact, without decorations that the model does not need.

### 3.7 Errors are results

A file that does not exist is a normal result, not a crash. The tool returns a short message: "not found: <path>. Use list_files to see what exists." Errors that the model can act on are part of the tool's design.

---

## 4. Gotchas

**4.1 Encoding.** Files in the project may be UTF-8, UTF-8 with a byte-order mark, or an older Windows encoding. Swedish letters in an older file may come out wrong if the wrong encoding is assumed. Decide one default, detect the byte-order mark, and say what happens otherwise.

**4.2 Line endings.** A file with `\r\n` endings and a search that splits on `\n` leaves a `\r` at the end of each line. This is the same issue as the BM25 tokenizer in Course 1. Trim line endings when returning lines.

**4.3 Very long lines.** A minified file has no line breaks, so a line limit does nothing. Limit by bytes as well as lines, and cut long lines with a marker.

**4.4 Searches that are too broad.** A search for `a` returns half the project. Require a minimum length, or warn in the output when a term matches too many lines.

**4.5 Symbolic links.** Following a link can leave the root. Decide whether links are followed; the safe default is to not follow them, and to say so.

**4.6 Race conditions.** A file can change between listing and reading. The tool may see a file that is gone. Treat "not found" at read time as normal.

---

## 5. The lab in outline

You will:

1. write the path-resolution rule and test it against escape attempts,
2. build `list_files`, `read_file`, and `search_text` with limits,
3. skip build and binary content, using a list from the config,
4. write descriptions and check how the model chooses,
5. measure the token cost of each tool's output,
6. test the boundary cases in a table.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Project root:** the folder that the tools are allowed to read.
- **Path resolution:** turning a path into its full absolute form, with links and `..` removed.
- **Binary file:** a file whose contents are not text, detected by its early bytes.
- **Output limit:** a maximum size or count for one tool result.
- **Truncation marker:** the message that says output was cut and how much.
- **Token cost:** the number of tokens a piece of text takes in the context.

---

## 7. Summary

- Three read-only tools: read one file, list a folder, search text.
- Paths are resolved first, then checked against the project root.
- Output is limited, and the model is told when it was cut.
- Build output and binary files are skipped by a list in the config.
- Descriptions say when to use each tool, and when not to.
- Output costs tokens, so search first and read the smallest useful unit.

Continue to [exercises.md](exercises.md).
