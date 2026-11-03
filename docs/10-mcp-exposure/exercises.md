# Lesson 10 — Exercises

This lesson's project is a **separate** console app from `LocalAgent` — create a new folder for it alongside your main project, e.g. `C:\Repos\LocalAgentMcp\`.

## Exercise 1 — Stand up a minimal MCP server

**Goal:** get the protocol plumbing working before porting real tools.

**Do**

1. **Create the new project and add the MCP SDK:**
   ```powershell
   cd C:\Repos
   dotnet new console -n LocalAgentMcp
   cd LocalAgentMcp
   dotnet add package ModelContextProtocol --prerelease
   dotnet add package Microsoft.Extensions.Hosting
   ```
2. **Replace `Program.cs`** with a minimal stdio-transport server exposing one trivial tool:
   ```csharp
   using Microsoft.Extensions.Hosting;
   using ModelContextProtocol.Server;
   using System.ComponentModel;

   var builder = Host.CreateApplicationBuilder(args);
   builder.Services
       .AddMcpServer()
       .WithStdioServerTransport()
       .WithTools<PingTool>();

   await builder.Build().RunAsync();

   [McpServerToolType]
   public static class PingTool
   {
       [McpServerTool, Description("Replies with pong. Use to check the server is reachable.")]
       public static string Ping() => "pong";
   }
   ```
   (Exact attribute/method names can shift between preview versions of the `ModelContextProtocol` package — check what IntelliSense offers for `[McpServerTool...]` if this doesn't compile as written, the shape stays the same.)
3. **Connect an MCP client to test it.** The simplest option is Claude Desktop: open its settings, find the MCP servers configuration file (on Windows, `%APPDATA%\Claude\claude_desktop_config.json`), and add an entry pointing at your built executable:
   ```json
   {
     "mcpServers": {
       "localagent": {
         "command": "dotnet",
         "args": ["run", "--project", "C:\\Repos\\LocalAgentMcp"]
       }
     }
   }
   ```
   Restart Claude Desktop, and check its MCP tool list (usually under a plug/tools icon in the chat input) for your `ping` tool.
4. Ask Claude Desktop to call your `ping` tool and confirm it gets back `"pong"`.

**Observe**
- What did you have to configure on the client side versus the server side? Which side defines the tool's name and description?

## Exercise 2 — Port your real tools

**Goal:** expose `search_notes` from your main `LocalAgent` project, without rewriting its logic.

**Do**

1. **Reference your `LocalAgent` project's `Rag` and `Tools` code from `LocalAgentMcp`** — either add a project reference if `LocalAgentMcp` can see `LocalAgent`'s `.csproj`, or copy `VectorStore.cs`, `Chunk.cs`, and the embedding-client setup over (a project reference is cleaner if you can manage it; copying is fine for this exercise).
2. **Wrap `NoteTools.SearchNotes`** as an MCP tool type, reusing the exact same `[Description]` text you tuned in Lesson 5 — this is the part to resist rewriting:
   ```csharp
   [McpServerToolType]
   public class NoteSearchMcpTool(NoteTools noteTools)
   {
       [McpServerTool, Description("Search the user's own notes for passages relevant to a question. Always call this FIRST for any question that might be about the user's notes, before answering from your own knowledge.")]
       public Task<string> SearchNotes(string query) => noteTools.SearchNotes(query);
   }
   ```
3. Register it with `.WithTools<NoteSearchMcpTool>()` alongside `PingTool`, index your notes folder at startup the same way `Program.cs` does in `LocalAgent`, and restart Claude Desktop.
4. In Claude Desktop, ask a question that should trigger your `search_notes` tool and check that it returns real results from your notes.

**Observe**
- Did you need to change `SearchNotes`'s actual logic at all, or only how it's registered and exposed?

## Exercise 3 — Deliberately do not expose the write tool

**Goal:** reason through the access-control implications of a remote caller.

**Do**

1. **Do not** wrap or register `ProposeCreateNote` (your Lesson 7 write tool) with the MCP server. Leave it only in your main `LocalAgent` console app.
2. Write down, concretely: if you did expose it over MCP, who could call it (any MCP client configured to connect to your server), from where (anywhere that client runs, not necessarily your machine), and what would need to be true — a confirmation mechanism *inside the MCP flow itself*, authentication on the server, no network exposure beyond localhost — before that would be safe.

**Discuss**
- Your console app's confirmation step from Lesson 7 runs in the same terminal the user is sitting at. If `propose_create_note` were called over MCP from a remote client like Claude Desktop, where would a confirmation prompt even need to live?
- What's genuinely new about the trust boundary here compared to Lesson 7's prompt-injection discussion — is it a new risk, or the same risk with a longer wire between the untrusted content and the action?
- If you later wanted to expose write access safely, what would the minimum safeguard be (e.g. an explicit allow-list, a required human-approval step built into the MCP tool-call flow)?
