# Module 4, Lesson 13 — Exercises: Sessions

Work in order. Each exercise ends with a **Checkpoint**.

**Before you start**
- **Where the code goes:** `Features/Sessions/` for resume, the picker, and their commands. The file format (`SessionHeader`, `SessionEvent`, `SessionWriter`, `SessionReader`) goes in `Shared/Sessions/`, because the loop writes events and the handoff in Lesson 15 reads them.
- Start from a clean git state, with Lesson 12 working.
- Sessions live in the `sessions` folder inside the main folder from Module 1, Lesson 1, never in the working folder. Write the path in your README.
- Start the chat `llama-server`.

---

## Exercise 0 — Lose a conversation on purpose

**Goal:** see the cost of having no sessions, so the lesson has a reason.

**Do**

1. Have a conversation of about five turns about the sandbox project, with a file read and a decision in it.
2. Close the app. Open it again. Ask the agent what you decided.
3. Note what it knows, and what it does not.

**Checkpoint:** a one-line note on what was lost.

---

## Exercise 1 — Define the event format

**Goal:** a file format that can be read without the app, and that can grow without being rewritten.

**Do**

1. Create `Shared/Sessions/SessionEvent.cs` with a record: a version number, a sequence number, a time, a kind (user, assistant, tool call, tool result, compaction, permission decision), and a payload.
2. Write the payload as a small set of fields per kind. For example, a tool call has a call identifier, a tool name, and the arguments as text.
3. Write the file header as the first line: a format name, a format version, the session identifier, and the project folder.
4. Write a short description of the format in `Shared/Sessions/FORMAT.md`, so a student can read a session file with a text editor.

Here is one event line, shown for orientation. The exact fields are yours to decide:

```json
{"v":1,"seq":4,"time":"2026-11-25T10:14:02Z","kind":"tool_result","callId":"c7","tool":"read_file","text":"  1: namespace Demo;\n  2: ...","bytes":412}
```

Start from this shell:

```csharp
using System.Text.Json.Nodes;

namespace LocalAgent.Shared.Sessions;

public static class EventKind
{
    public const string User = "user";
    public const string Assistant = "assistant";
    public const string ToolCall = "tool_call";
    public const string ToolResult = "tool_result";
    public const string Compaction = "compaction";
    public const string Permission = "permission";
}

// The first line of every session file.
public record SessionHeader(string Format, int FormatVersion, string SessionId, string ProjectFolder);

// One line per event. Payload holds the fields for this kind (step 2), for example
// callId, tool and text for a tool result. Kept as JSON so new fields need no new type.
public record SessionEvent(int V, long Seq, DateTimeOffset Time, string Kind, JsonObject Payload);
```

The example line above writes the payload fields next to the others, on one level. Decide whether your file keeps them flat, as in the example, or under a `payload` key, and write the choice in `FORMAT.md`.

**Read this carefully.** The `bytes` field is a size, not the content. The rule of Section 3.4 says that large content is referenced, not copied. Decide, for each kind of event, whether its text is stored or only its size and fingerprint.

**Checkpoint:** you have a header line and at least four event kinds, and `FORMAT.md` explains each field in one sentence.

---

## Exercise 2 — Write append-only

**Goal:** each event is added to the end of the file, as one line, and a crash cannot damage earlier lines.

**Do**

1. Create `Shared/Sessions/SessionWriter.cs`. It opens the session file in append mode, writes one event as one line, and flushes it.
2. Use one writer per session. When a session opens, check whether a lock file exists. If it does, refuse to open the session, and say another window may be using it.
3. Write the event before the app acts on it where possible. A tool result is written before it is sent to the model, so a crash after the send can be recovered.

Start from this shell:

```csharp
namespace LocalAgent.Shared.Sessions;

public sealed class SessionWriter : IDisposable
{
    // Creates the lock file next to the session file. Throws a clear error if it already exists:
    // another window may be using the session (step 2).
    public SessionWriter(string sessionPath, SessionHeader? headerForNewFile)
    {
        throw new NotImplementedException(); // students write this
    }

    // One event, one line, a line break at the end, then flush.
    // Returns the error text if the write failed, or null. The event stays in memory either way.
    public string? Append(SessionEvent sessionEvent)
    {
        throw new NotImplementedException(); // students write this
    }

    // Closes the file and removes the lock file.
    public void Dispose()
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** The write must end with a line break, and must be flushed. A line without its break is the damage a crash leaves, and the reader must recognise it.

**Checkpoint**
- After five turns, the session file has one header and the events in order, one per line.
- Kill the app in the middle of an answer, open the file, and confirm that the earlier lines are complete.

---

## Exercise 3 — Read and skip damaged lines

**Goal:** a session can be read even when one line is damaged, and the damage is reported.

**Do**

1. Create `Shared/Sessions/SessionReader.cs`. It reads the file line by line, parses each line, and keeps the events that parse.
2. A line that does not parse is skipped, and counted. At the end, report "N lines could not be read" with the line numbers.
3. The header is checked first. A header with a newer format version is refused, with a message that names both versions.

Start from this shell:

```csharp
namespace LocalAgent.Shared.Sessions;

public record SessionFile(SessionHeader Header, IReadOnlyList<SessionEvent> Events, IReadOnlyList<int> DamagedLines);

public static class SessionReader
{
    public const int SupportedFormatVersion = 1;

    // Reads line by line. Line 1 is the header: a newer FormatVersion is refused with a message
    // that names both versions. A line that does not parse is skipped, and its number recorded.
    public static (SessionFile? File, string? Error) Read(string sessionPath)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Checkpoint:** edit one line of a session file to be invalid JSON. Opening the session shows the warning with the line number, and the other events are present.

---

## Exercise 4 — Rebuild the history and check it

**Goal:** resuming produces a history that is valid for the model, or says why it is not.

**Do**

1. Create `Features/Sessions/Resume.cs`. It takes the events and builds the working history, in the same order, with the same roles as during the session.
2. Check validity: every tool call has a result, and every result has a call. For a call with no result, add a result that says "not recorded: the session ended before the tool finished", so the history can still be sent. Print a line saying this was done.
3. Do not replay tool calls. A resume shows the record, and the next question gets fresh results.

Start from this shell. Exercise 7 adds the compaction rule:

```csharp
using Microsoft.Extensions.AI;

namespace LocalAgent.Features.Sessions;

public static class Resume
{
    // Builds the working history from the events: same order, same roles. Never runs a tool.
    // A call with no result gets the "not recorded" result from step 2, and its call id is
    // listed in Repaired so the caller can print a line about it.
    public static (List<ChatMessage> History, IReadOnlyList<string> Repaired) Build(IReadOnlyList<SessionEvent> events)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Read this carefully.** Never run a tool again during a resume. A write that was approved last week must not happen again because the session was opened. The resume is a reading, not a repeat.

**Checkpoint**
- Resume the session from Exercise 2. The agent knows the decisions from the earlier turns.
- A session where you killed the app mid-tool-call resumes, with the missing result marked as not recorded, and the next question works.

---

## Exercise 5 — The picker

**Goal:** the user can choose a session from a list, with enough shown to choose.

**Do**

1. Create `Features/Sessions/SessionIndex.cs` that lists sessions in the sessions folder. For each, read only the header and the first user message, not the whole file, so the list is fast.
2. Show the date of the last event, the first user message cut to one line, the number of user turns, and the project folder. Sort by the last event's time.
3. Add `/resume` with no argument to show the picker, and `/resume <number>` to open one. Sessions with no user message are not listed.
4. Check the time of the last event, not the file's modification time, for the sort order (Gotcha 4.4).

Start from this shell:

```csharp
namespace LocalAgent.Features.Sessions;

public record SessionSummary(string Path, DateTimeOffset LastEvent, string FirstUserMessage, int UserTurns,
    string ProjectFolder);

public static class SessionIndex
{
    // One summary per session file that has a user message, newest LastEvent first.
    // Reads the header, the first user message, and the last line, not every event (step 1).
    // Counting UserTurns needs a pass over the lines; parse only the "kind" field for that.
    public static IReadOnlyList<SessionSummary> List(string sessionsFolder)
    {
        throw new NotImplementedException(); // students write this
    }
}
```

**Checkpoint:** create three sessions with different first messages, and check that the picker lists them in the right order with the right first messages.

---

## Exercise 6 — Delete with confirmation

**Goal:** a session is deleted only after the user confirms its details.

**Do**

1. Add `/sessions delete <number>`. It shows the date, the turn count, and the first message, and asks for the word `yes`.
2. Anything other than `yes` cancels, and the file stays.
3. After deletion, print the path that was removed, so the user has a record.

**Checkpoint:** choose a session to delete, press Enter at the prompt, and confirm the file still exists. Then type `yes` and confirm it is gone.

---

## Exercise 7 — Compaction and the transcript

**Goal:** a compaction (Lesson 11) writes a compaction event, keeps the old events in the file, and resume uses the latest summary.

**Do**

1. When compaction runs, write a compaction event that holds the summary text and the sequence number of the last message it covers.
2. On resume, start from the latest compaction event: use its summary as the first message, then the events after it. The events before it stay in the file, but are not loaded.
3. Check that a full read of the file still shows every original event, for the user and for debugging.

**Checkpoint:** compact a session, close the app, resume it. The history starts with the summary. Read the file in an editor and confirm that the original messages are still there.

---

## Exercise 8 — Test the edge cases

Work through this table. Record a result for each row.

| Test | How to produce it | Expected result |
|---|---|---|
| Two windows open the same session | open one session in two windows | the second refuses with a clear message |
| Session file locked by another program | open the file in an exclusive mode in another program | the write error is reported, the event is not lost from memory |
| Damaged line in the middle | invalid JSON in line 5 | warning with line number, other lines used |
| Newer format version | change the header version to 99 | refused, both versions named |
| Call without result after a crash | kill the app during a tool | resumed with "not recorded" result |
| Session with no user message | start and close without asking | not listed in the picker |
| Two sessions with the same first message | create two identical starts | both listed, distinguished by date |
| Delete, then answer no | start deleting, answer no | file kept |
| Swedish letters | a first message with `å`, `ä`, `ö` | shown correctly in the picker and the resume |
| Very long session | a session with several hundred turns | resume loads only from the latest compaction, and says so |

**Checkpoint:** every row has a written result, and each surprise is fixed or written down.

---

## Observe (whole lesson)

1. What did a resumed session get wrong that a new session did not? What would you tell a user to do about it?
2. The session file is readable without the app. What does that give a user, and what risk does it bring?
3. Compare the working history after compaction with the session file. Which one should a debugging student read, and why?

---

## Discuss

- A session is a record of what the agent did. Who owns that record, and what should happen to it when the user deletes the project?
- Append-only files are safe to crash, but they grow. Where would you put the limit, and what would the user see when it is reached?
- A resumed session may describe files that have changed. What should the agent say, and what should it avoid saying?

---

## Stretch

- Export a session to a Markdown file for sharing, with secrets removed by the Lesson 12 check.
- Add a search across sessions for a word, with the result showing the session and the event.
- Compress old sessions, and decide what the picker must do with a compressed one.

---

## Time check

If you are past the planned time, finish Exercises 2, 3, and 4. Append-only writing, damage-tolerant reading, and a validated resume are what make a session trustworthy.
