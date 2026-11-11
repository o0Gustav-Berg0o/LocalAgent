# Lesson 1 — Exercises

All file paths below are relative to your `LocalAgent` project root (the folder with `LocalAgent.csproj`) from Lesson 0. Both `llama-server` windows from Lesson 0 need to stay running for every exercise in this lesson.

## Exercise 1 — Two trivial tools

**Goal:** get a tool call working end-to-end before anything RAG-related exists.

**Do**

- [ ] **Add the function-calling package**, which gives you `AIFunctionFactory` to turn a plain C# method into a tool the model can call:
   ```powershell
   dotnet add package Microsoft.Extensions.AI.Abstractions
   ```
- [ ] **Create a new file `Tools/UtilityTools.cs`** (create the `Tools` folder if it doesn't exist) with two methods:
   ```csharp
   using System.ComponentModel;

   namespace LocalAgent.Tools;

   public static class UtilityTools
   {
       [Description("Get the current local date and time. Use this whenever the user asks what day, date, or time it is.")]
       public static string GetDateTime()
           => DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");

       [Description("Evaluate a simple arithmetic expression, e.g. \"17 * 23\" or \"(4 + 5) / 2\". Use this for any calculation instead of computing it yourself.")]
       public static string Calculate(
           [Description("The arithmetic expression to evaluate, using + - * / and parentheses only.")]
           string expression)
       {
           try
           {
               // DataTable.Compute is a quick way to evaluate a simple arithmetic string without
               // writing your own parser — it's not built for this, but it's good enough here.
               var result = new System.Data.DataTable().Compute(expression, null);
               return result.ToString() ?? "Error: could not compute a result.";
           }
           catch (Exception ex)
           {
               // The model reads this message, not a developer — keep it short and return it as
               // the tool result instead of letting the exception propagate and crash the loop.
               return $"Error: {ex.Message}";
           }
       }
   }
   ```
   Notice the `[Description]` on `Calculate` itself says *when* to use it ("instead of computing it yourself"), not just what it does — that phrasing matters more than it looks like it should; Lesson 5 is entirely about why.
- [ ] **Turn the methods into tools** in `Program.cs`. Placement: right after you build your `chatClient` (from Lesson 0 Exercise 4) and before the one-shot `client.GetResponseAsync("Hello")` call you wrote there — you'll replace that one-shot call entirely in Exercise 2, but for now the `tools` array just needs to exist before anything tries to use it:
   ```csharp
   using Microsoft.Extensions.AI;
   using LocalAgent.Tools;

   AIFunction[] tools =
   [
       AIFunctionFactory.Create(UtilityTools.GetDateTime),
       AIFunctionFactory.Create(UtilityTools.Calculate),
   ];
   ```
- [ ] **Print the generated schema** before you do anything else with it, just to see what the model actually receives. Placement: directly below the `tools` declaration you just added, still before the old one-shot call:
   ```csharp
   foreach (var tool in tools)
       Console.WriteLine(tool.JsonSchema);
   ```
   Run with `dotnet run` and look at the printed JSON.

**Observe**
- In the printed schema, where does the text from your `[Description]` attributes end up? Is it exactly what you wrote, word for word?

## Exercise 2 — Hand-write the loop

**Goal:** implement the agent loop yourself, without a framework's automatic function-invocation helper, so every step stays visible.

**Do**

- [ ] **Create `Agent/AgentLoop.cs`** (create the `Agent` folder if needed) with a class that holds the conversation history and runs the loop:
   ```csharp
   using Microsoft.Extensions.AI;

   namespace LocalAgent.Agent;

   public class AgentLoop(IChatClient client, IList<AITool> tools)
   {
       // The whole conversation so far, resent in full on every call — the model itself
       // remembers nothing between requests (Lesson 6 covers this in depth).
       private readonly List<ChatMessage> _history = [];

       // A hard ceiling so a confused model can't call tools forever.
       private const int MaxToolIterations = 6;

       public async Task<string> AskAsync(string userInput)
       {
           _history.Add(new ChatMessage(ChatRole.User, userInput));

           for (int i = 0; i < MaxToolIterations; i++)
           {
               Console.WriteLine($"[llm] call {i + 1} — {_history.Count} messages in history");

               // Every iteration resends the entire history — see the comment on _history above.
               var response = await client.GetResponseAsync(
                   _history,
                   new ChatOptions { Tools = tools });

               _history.AddMessages(response);

               // The model only ever *requests* a tool call via FunctionCallContent — it never
               // executes anything itself. Your code below decides whether and how to run it.
               var calls = response.Messages
                   .SelectMany(m => m.Contents.OfType<FunctionCallContent>())
                   .ToList();

               // No tool requests means the model considers itself done — return its text and exit.
               if (calls.Count == 0)
                   return response.Text;

               foreach (var call in calls)
               {
                   string result;
                   try
                   {
                       var tool = tools.OfType<AIFunction>().First(t => t.Name == call.Name);
                       var toolResult = await tool.InvokeAsync(new AIFunctionArguments(call.Arguments));
                       result = toolResult?.ToString() ?? "";
                   }
                   catch (Exception ex)
                   {
                       // Turn a thrown exception into a result string instead of letting it crash
                       // the loop, so the model gets a chance to see what went wrong and recover.
                       result = $"Error: {ex.Message}";
                   }

                   // Only the console trace line is truncated — the full result still goes to the model below.
                   var truncated = result.Length > 600 ? result[..600] + "…" : result;
                   Console.WriteLine($"[tool] {call.Name}({string.Join(", ", call.Arguments?.Select(a => $"{a.Key}={a.Value}") ?? [])}) -> {truncated}");

                   // Feed the result back as a tool-role message so the next [llm] call above sees it.
                   _history.Add(new ChatMessage(ChatRole.Tool,
                       [new FunctionResultContent(call.CallId, result)]));
               }
           }

           // Hit MaxToolIterations without the model ever stopping on its own — bail out
           // rather than looping forever.
           return "Sorry, I couldn't finish that after several tool calls — something may be stuck in a loop.";
       }
   }
   ```
   (Exact method names on `IChatClient`/`AIFunction` can differ slightly between `Microsoft.Extensions.AI` prerelease versions — if something doesn't compile, check IntelliSense on `client.` and `tool.` for the closest equivalent; the shape of the loop stays the same either way.)
- [ ] **Wire it up in `Program.cs`**, replacing your Lesson 0 one-shot call:
   ```csharp
   var loop = new AgentLoop(chatClient, tools);

   while (true)
   {
       Console.Write("You> ");
       var input = Console.ReadLine();
       if (string.IsNullOrWhiteSpace(input)) break;

       var answer = await loop.AskAsync(input);
       Console.WriteLine($"Agent> {answer}");
   }
   ```
- [ ] **Run it**: `dotnet run` from the project root, then try the questions in Observe below.

**Observe**
- Ask: *"What's today's date, and what is 17 * 23?"* — watch the `[llm]` and `[tool]` trace lines. How many model calls did it take? Why more than one?
- Ask something that needs no tool at all: *"Hi, who are you?"* — did the model call a tool anyway? Should it have?

**Discuss**
- What would happen if a tool call result were 50,000 characters long? Where would that text go, and what would it cost on the next `[llm]` call?
- Why does catching tool exceptions and returning them as a `"Error: ..."` string matter more here than it would in a typical REST API?
