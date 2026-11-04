# Lesson 5 — Exercises

Both `llama-server` windows must still be running. You're editing `Tools/NoteTools.cs` from Lesson 4 for every exercise here.

## Exercise 1 — Starve the description

**Goal:** see the effect of a vague tool description directly.

**Do**

- [ ] In `Tools/NoteTools.cs`, temporarily change the `[Description]` attributes on `SearchNotes`:
   ```csharp
   [Description("Search")]
   public async Task<string> SearchNotes(
       [Description("q")] string query)
   ```
   (Comment out or save a copy of the original text first — you'll restore it exactly in Exercise 2.)
- [ ] `dotnet run`, and ask three questions whose answers are clearly in your notes (reuse ones from Lesson 4 that worked reliably before this change).

**Observe**
- Is the tool still called? With sensible arguments, or does the model hesitate, skip it, or call it with odd/invented arguments? Check the `[tool]` trace line for exactly what argument value it sent.

## Exercise 2 — Add overlapping, confusing tools

**Goal:** see what happens when the model has to choose between similar options.

**Do**

- [ ] **Restore the original, well-written description** on `SearchNotes` (from Lesson 4).
- [ ] **Add three more methods to `NoteTools.cs`** that all just call `SearchNotes` internally, with different names and overlapping-but-not-identical descriptions:
   ```csharp
   [Description("Find documents related to a topic in the user's files.")]
   public Task<string> FindDocuments(string topic) => SearchNotes(topic);

   [Description("Look up information the user has stored.")]
   public Task<string> LookupInfo(string query) => SearchNotes(query);

   [Description("Query the user's personal knowledge base for relevant content.")]
   public Task<string> QueryKnowledgeBase(string query) => SearchNotes(query);
   ```
- [ ] **Register all four** in `Program.cs`'s tool list (`AIFunctionFactory.Create(noteTools.FindDocuments)`, etc.) and `dotnet run`.
- [ ] Ask the same three questions from Exercise 1.

**Observe**
- Which tool does the model actually pick, per the trace? Does it sometimes call more than one, or pick the "wrong" one?

## Exercise 3 — Remove the system prompt's directive

**Goal:** isolate the effect of instruction *placement* (prompt vs. tool description) from instruction *content*.

**Do**

- [ ] **Remove the three extra tools** you added in Exercise 2 from the tool list in `Program.cs` (keep the methods in the file if you want, just don't register them) — you're back to just `search_notes` with its good description.
- [ ] **In your `SystemPrompt` field in `AgentLoop.cs`** (added in Lesson 4), delete the sentence telling the model to call `search_notes` first.
- [ ] `dotnet run` and ask the same questions again.

**Observe**
- Does the model skip searching more often now, even though `search_notes`'s own description is still good?

**Discuss**
- Why is "few, clear tools" a hard design constraint for small models rather than a style preference?
- How would you expect an 8B or 14B model, or a large cloud model, to behave differently on these same exercises?
- If you genuinely needed 15 tools in a real application, how would you structure them to stay usable by a small model (e.g. grouping, routing, or a larger model for tool selection)?

When you're done, restore `search_notes` to exactly its Lesson 4 state (good description, registered alone, system prompt directive back in place) before moving on.
