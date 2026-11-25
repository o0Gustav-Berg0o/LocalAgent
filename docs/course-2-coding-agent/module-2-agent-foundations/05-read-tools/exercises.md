# Module 2, Lesson 5 — Exercises: Read-Only Tools

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/ReadFiles/`. `ProjectPaths` goes in `Shared/Files/`, because every file tool after this lesson uses it.
- Start from a clean git state, with Lesson 4 working.
- Create a sample project folder for testing, separate from your notes. For example `C:\Repos\sandbox` with a few small code files, one folder named `bin` with a large file in it, and one binary file (any `.png` or `.exe` you trust). You will use it in every exercise.
- Start the chat `llama-server`.

---

## Exercise 0 — Watch the model search without tools

**Goal:** see what the model does when it cannot look at files, so you have a baseline for Exercise 6.

**Do**

1. Start the app, and ask: "Where is the function that parses the config file in my sandbox project?"
2. Write down the answer. Note whether it names a real file, and whether it admits it cannot see the project.
3. Ask the same question again, and write down whether the answer changes.

**Observe**
- Does the model invent a file name? That is the reason tools exist. Keep the invented answer for comparison.

**Checkpoint:** you have one written answer, and you have noted whether it is grounded.

---

## Exercise 1 — Resolve paths and reject escapes

**Goal:** a single function decides whether a path is inside the project root. No tool calls it without going through it.

**Do**

1. Create `Shared/Files/ProjectPaths.cs` with a static method that takes the project root and a path given by the model, and returns either the full resolved path, or a refusal with a reason.
2. The method must:
   - combine the path with the root if it is relative,
   - resolve `..` and links to the full path,
   - compare the result with the root, case-insensitively, and accept only paths that start with the root followed by a separator,
   - refuse everything else, with a reason the model can read.

Here is the shape. Fill in the details yourself, and note what each line protects against:

```csharp
namespace LocalAgent.Shared.Files;

public static class ProjectPaths
{
    public static bool TryResolve(string root, string requested, out string fullPath, out string reason)
    {
        // 1. Combine with the root if relative. Absolute paths are still checked below.
        // 2. Resolve to the full path (removes ".." and normalises separators).
        // 3. Compare with the root. Use a case-insensitive comparison on Windows.
        // 4. Require the root plus a separator as the prefix, so "C:\proj2" is not
        //    accepted for root "C:\proj".
        // 5. Refuse with a reason the model can act on.
        throw new NotImplementedException();
    }
}
```

**Read this carefully.** Step 4 is the one students skip. Without the separator, a root of `C:\proj` accepts `C:\proj2\secret.txt`, because the text starts with the root. Write a test for it.

**Checkpoint:** run the method with these inputs and confirm the results.

| Requested | Expected |
|---|---|
| `src\Program.cs` | accepted |
| `..\secret.txt` | refused |
| `C:\Windows\win.ini` | refused |
| `C:\sandbox2\x.txt` (root `C:\sandbox`) | refused |
| `SRC\program.cs` on Windows | accepted |

---

## Exercise 2 — List files with limits

**Goal:** `list_files` returns names in a folder, skipping what should be skipped, and says when the list was cut.

**Do**

1. Create `Features/ReadFiles/ListFilesTool.cs`. Its input is a folder path (default: the root) and an optional pattern such as `*.cs`.
2. Use `ProjectPaths.TryResolve` on the folder. Refuse if it fails.
3. Skip folders named in the config's skip list (`bin`, `obj`, `.git`, `.vs`, `node_modules`). Skip files that are binary (write the detector in Exercise 4, and call it here as a stub for now).
4. Limit the output to 200 entries. If there are more, add a final line: `... N more entries not shown. Narrow the pattern or list a subfolder.`
5. Return one name per line, relative to the root, with a trailing `/` for folders.

Start from this shell. A tool is a class with one method, registered with `AIFunctionFactory` in Exercise 6:

```csharp
using System.ComponentModel;

namespace LocalAgent.Features.ReadFiles;

public sealed class ListFilesTool(string root, IReadOnlyList<string> skipFolders)
{
    public const int MaxEntries = 200;

    [Description("Written in Exercise 6.")]
    public string ListFiles(
        [Description("Folder relative to the project root. Empty for the root.")] string folder = "",
        [Description("Optional file pattern, for example *.cs.")] string? pattern = null)
    {
        // 1. ProjectPaths.TryResolve(root, folder, ...). Refuse with the reason if it fails.
        // 2. Walk the folder. Skip skipFolders and binary files (BinaryDetector, Exercise 4).
        // 3. One name per line, relative to the root, with "/" after a folder name.
        // 4. Stop at MaxEntries, and add the cut message from step 4.
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The model reads the list as text. A folder with 5000 names costs tokens and says little. The limit and the message are the feature.

**Checkpoint**
- `list_files` on the sandbox root shows the source files, not the `bin` folder.
- A pattern such as `*.cs` narrows the list.
- A folder with more than 200 entries shows the cut message at the end.

---

## Exercise 3 — Read a file with a range and a byte limit

**Goal:** `read_file` returns a range of lines from one file, and refuses files that are too large or binary.

**Do**

1. Create `Features/ReadFiles/ReadFileTool.cs`. Inputs: a path, an optional start line (1-based, default 1), and an optional count (default 200).
2. Resolve the path. Refuse if it fails.
3. Read only the lines needed, not the whole file, if the file is large. A simple approach is to stream lines and stop after the count.
4. Enforce a byte limit of 32 KB on the returned text. If the limit is hit, cut at a line boundary, and say so.
5. Number each returned line with its line number, so the model can refer to exact lines later. Format: `  42: text`.
6. If the file has more lines after the returned range, add a final line: `... file has N lines in total. Use start_line to read more.`

Start from this shell:

```csharp
using System.ComponentModel;

namespace LocalAgent.Features.ReadFiles;

public sealed class ReadFileTool(string root)
{
    public const int MaxBytes = 32 * 1024;

    [Description("Written in Exercise 6.")]
    public string ReadFile(
        [Description("File path relative to the project root.")] string path,
        [Description("First line to return, starting at 1.")] int startLine = 1,
        [Description("Number of lines to return.")] int count = 200)
    {
        // 1. Resolve, and refuse as ListFiles does. Refuse a binary file (Exercise 4).
        // 2. Stream the lines (File.ReadLines). Skip to startLine, stop after count.
        // 3. Number each line: "  42: text". Stop at MaxBytes, at a line boundary, and say so.
        // 4. If the file has more lines after the range, add the total-lines message (step 6).
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Line numbers must match the file on disk. If you strip line endings or skip blank lines, the numbers drift. Test with a file that has blank lines in the middle.

**Checkpoint**
- Reading lines 40 to 50 of a sample file returns exactly those lines, with the right numbers.
- A file of 10,000 lines returns 200 lines and the total-line message.
- A file with a very long single line is cut at the byte limit, and the message says so.

---

## Exercise 4 — Detect binary files

**Goal:** a binary file is refused with a clear message, and never returned as text.

**Do**

1. Write `Features/ReadFiles/BinaryDetector.cs` with a method that reads the first 8 KB of a file and returns true if it contains a zero byte, or if a large share of bytes are not valid text. Keep the rule simple and written in a comment.
2. Call it from `ReadFileTool` and `ListFilesTool`. A binary file is refused with: `<name> looks like a binary file, so it was not read.`
3. Add one test for each: a text file with no zero bytes, a file with a zero byte, a file of random bytes.

Start from this shell:

```csharp
namespace LocalAgent.Features.ReadFiles;

public static class BinaryDetector
{
    public const int SampleBytes = 8 * 1024;

    // The rule, in one sentence: (write it here).
    // Reads at most SampleBytes from the start of the file. A zero byte means binary.
    // Otherwise, decide which share of non-text bytes counts as binary, and write the number here.
    public static bool LooksBinary(string fullPath)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Checking the extension alone is not enough. A file named `image.txt` can hold binary data, and a `.cs` file can hold a zero byte by mistake. Check the content.

**Checkpoint:** the three test files give the three expected results, and a binary file in the sandbox is refused by both tools.

---

## Exercise 5 — Search text across the project

**Goal:** `search_text` finds lines containing a term, with file and line number, and limits the results.

**Do**

1. Create `Features/ReadFiles/SearchTextTool.cs`. Inputs: the text to find, and an optional folder to search in (default: the root).
2. Walk the project, skip the skip list, skip binary files, and check each line for the text. Use a case-insensitive comparison by default.
3. Return at most 50 matches. Each match is one line: `path:line: text`, with the text trimmed and cut at 160 characters.
4. If more than 50 matches exist, add: `... N more matches. Narrow the search text or search a subfolder.`
5. Refuse a search text shorter than three characters, with a message that asks for a longer term. This protects against searches like `a`, which match almost everything.

Start from this shell. If the folder walk in `ListFilesTool` can be shared, move it to a small helper and call it from both:

```csharp
using System.ComponentModel;

namespace LocalAgent.Features.ReadFiles;

public sealed class SearchTextTool(string root, IReadOnlyList<string> skipFolders, int maxMatches, int minLength)
{
    [Description("Written in Exercise 6.")]
    public string SearchText(
        [Description("The text to find. At least three characters.")] string text,
        [Description("Folder to search, relative to the project root. Empty for the root.")] string folder = "")
    {
        // 1. Refuse text shorter than minLength (step 5).
        // 2. Resolve the folder. Walk it, skipping skipFolders and binary files.
        // 3. Each match: "path:line: text", trimmed and cut at 160 characters.
        // 4. Stop at maxMatches, and add the cut message with the count (step 4).
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The 50-match limit protects the context. The three-character minimum protects the search from being useless. Both are numbers you can change, so put them in the config and say so in the description.

**Checkpoint**
- Searching for a name that exists in one file returns that file, with the right line number.
- Searching for `a` is refused with the minimum-length message.
- A search that matches more than 50 lines shows the cut message.

---

## Exercise 6 — Write descriptions and check the choice

**Goal:** the model picks the right tool for the task, because the descriptions say when to use each one.

**Do**

1. Write a description for each tool, following the three parts from the README (what it returns, when to use it, when not to). Keep each description under 60 words, because the descriptions are sent with every request and cost tokens every turn.
2. Register the three tools in `Program.cs`, with names in snake_case: `list_files`, `read_file`, `search_text`.
3. Ask three questions that need different tools:
   - "What files are in the Tools folder?" (expect `list_files`)
   - "Show me lines 1 to 30 of Program.cs." (expect `read_file`)
   - "Where is the word 'config' used?" (expect `search_text`)
4. Read the `[tool]` trace for each. Did the model pick the expected tool? Did it pick it on the first try?
5. Change one description on purpose: remove the "when not to use" part from `search_text`. Repeat question 3. Restore the description after the test.

**Observe**
- Did the model call the wrong tool once? Which description would have prevented it?
- Did the model read a whole file when a search would have done? What did that cost in tokens?

**Checkpoint:** each of the three questions picks the expected tool, and you have a written note on the description test.

---

## Exercise 7 — Measure the token cost

**Goal:** know how many tokens each tool's output costs, so limits are chosen with numbers, not guesses.

**Do**

1. For each tool, run a typical call and count the characters of its output. Use the sandbox project and a realistic file.
2. Convert characters to an estimate of tokens: divide the character count by 4, and say that this is an estimate for English-like text. Swedish and code may differ; note that.
3. Write a table: tool, example call, characters, estimated tokens.
4. Decide: is the 32 KB read limit too large? A rough rule: keep one tool result under about 2,000 to 3,000 tokens for a local model with a small context. Adjust a limit if the numbers say so, and write why.

**Checkpoint:** the table has three rows, and you have one decision about a limit, with the number behind it.

---

## Exercise 8 — Test the boundaries

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Path escape with `..` | ask for `..\..\x.txt` | refused, reason shown |
| Absolute path outside root | ask for a file in `C:\Windows` | refused |
| Sibling folder with same prefix | root `sandbox`, request `sandbox2\x` | refused |
| Missing file | ask for a file that does not exist | "not found" with a hint, no crash |
| Binary file | read a `.png` | refused as binary |
| Very long line | a file with a 100 KB line | cut at byte limit, marker shown |
| Huge folder | list a folder with 500 files | 200 shown, cut message |
| Search too short | search `ab` | refused, minimum length message |
| Search too broad | search `the` in a large project | 50 shown, cut message |
| Swedish letters in a file | search for `ö` in a UTF-8 file | match found, text correct |

**Checkpoint:** every row has a written result, and each failure is fixed or explained.

---

## Observe (whole lesson)

1. Which limit did you find most important, and what would break first without it?
2. Name one way a path could still escape after Exercise 1. Would your test catch it?
3. Compare Exercise 0 with Exercise 6. What changed in the model's answers, and what changed in the tools?

---

## Discuss

- The model can read anything the root allows. Who decides what the root is, and what should happen if the user picks their home folder?
- Why is a truncation message better than silently returning less? What would the model do without it?
- Search is cheap for the model, and reading is expensive. How does this shape the way a good agent works?

---

## Stretch

- Add an `--ignore` option, so the user can add a folder to the skip list for one session only.
- Add a `search_text` option for regular expressions. What new risks does it add, and what limit would you put on it?
- Add a `file_info` tool that returns size and line count without the content. When is it worth the extra tool?

---

## Time check

If you are past the planned time, finish Exercises 1, 3, and 5. Path checking and output limits are the two protections this lesson exists to provide.
