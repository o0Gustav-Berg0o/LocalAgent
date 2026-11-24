# Module 2, Lesson 3 — One Provider Interface and a Config File

**Time:** about 3 hours (60 min reading, 90 min lab, 30 min quiz and reflection).

**Prerequisites:**
- Module 1, Lesson 1: the config file exists and is read on start.
- Module 2, Lesson 2: the loop streams its answers.

**Quiz:** [quiz.md](quiz.md)

---

## 1. Learning objectives

By the end of this lesson you can:

1. Explain why the rest of the app must not know which model server it talks to.
2. Build one provider interface that the loop calls, with a local implementation behind it.
3. Read the chat and embedding endpoints, model names, and context size from the config file, and remove every hardcoded copy.
4. Validate the config on load, and say what the app does for each kind of invalid value.
5. Build a check that tells the user which setting is wrong, instead of failing with a stack trace.
6. Explain why the context size is a setting of the model server as well as of the app, and what happens when the two disagree.

---

## 2. Why this lesson exists

Up to now the app creates its client in one place, in `Program.cs`, with the endpoint and model name written into the code. That works while one developer runs one server. It breaks as soon as you want to:

- run the same app against a different model,
- run it on a machine where the server has another port,
- test the loop without a server at all,
- add a second kind of provider later.

Every time a value is hardcoded, changing it means editing code, rebuilding, and hoping no other copy is left behind. Lesson 1 found three such copies of the notes folder path. This lesson removes the copies and gives the rest of the app one answer to "which model am I talking to?".

---

## 3. Concepts

### 3.1 The seam

A **seam** is a place where you can change one part without touching the rest. The loop talks to an interface, not to a concrete client. The interface has a small number of methods: send messages, receive a response or a stream. Behind it, a class talks to `llama-server`.

```text
AgentLoop  ──►  IChatProvider  ──►  LocalLlamaProvider  ──►  llama-server (port 8080)
                    │
                    └─────────────►  FakeProvider (for tests, no server needed)
```

Two rules keep the seam useful:

- **The loop does not import the server library.** If `AgentLoop.cs` mentions the server client, the seam is not there.
- **The provider does not know about tools or the history format beyond what the library requires.** Tool selection and history belong to the loop.

### 3.2 What a provider needs to know

A provider needs:

| Setting | What it is | Where it comes from |
|---|---|---|
| endpoint | the address of the server, including `/v1` if the server uses it | config file |
| model name | the name the server expects in each request | config file |
| context size | how many tokens the model accepts | config file, and checked against the server |
| timeout | how long a request may take before it fails | config file, with a default |

The provider does not store the user's messages, does not choose tools, and does not decide what to show. Those are the loop's jobs.

### 3.3 Two places hold the context size

This is a subtle point, and a common source of bugs.

- The **model server** has a context size. It is set when the server starts (a command-line option). Requests longer than this fail, or are cut off, depending on the server.
- The **app** has a context size setting. It decides when to compact the history (Module 4).

If the app thinks the context is 8192 tokens and the server was started with 4096, the app will send requests the server cannot handle, and the failure will look like a model error. The config file holds the app's value. The validation step in this lesson checks that the two are at least plausible together, and the lab asks you to make the server's value visible to the user.

### 3.4 Validation on load

A config file is text that a person edits. People make mistakes: a missing bracket, a port written as a word, an endpoint without `http://`. The loader must do three things for every field:

1. **Accept** a valid value.
2. **Replace** an invalid value with the default, and record a warning that names the field and the reason.
3. **Never crash** because of a bad value. The app must start, show the warnings, and let the user fix the file or run `/settings`.

Keep the rules in one place. A rule that exists in two places will eventually disagree with itself.

### 3.5 Errors the user can act on

When the server is not running, the error message should say:

- which endpoint failed,
- what kind of failure it was (refused, timeout, not found, bad response),
- the most likely fix.

"Connection refused on http://localhost:8080/v1. Is llama-server running on port 8080?" is useful. "HttpRequestException" is not. This is the same principle as the endpoint check in Module 1, applied to every request, not only at setup.

### 3.6 Timeouts

A local model can take minutes on a long answer. A short default timeout will cut off a correct answer and make the app look broken. A long timeout means a hung server holds the user for minutes. Choose a value, make it a setting, and explain the trade-off in the config file's comments. JSON does not allow comments, so put the explanation in the README and in the settings menu's help text.

### 3.7 Why not an API key

This course is local only. The provider interface could later support a remote API, and that is a deliberate choice for Module 7. For now the config file holds no key field. If a key field appears in a file, the validation step should flag it as unexpected and say that keys belong in environment variables, not in files.

---

## 4. Gotchas

**4.1 A trailing slash.** `http://localhost:8080/v1/` and `http://localhost:8080/v1` may produce different request paths when joined with a relative path. Normalise the endpoint once, when you load it.

**4.2 Localhost versus 127.0.0.1.** On some Windows setups, `localhost` resolves to IPv6 first, and a server listening only on IPv4 looks unreachable. If one works and the other does not, try the other before assuming the server is down.

**4.3 Model names are not file names.** The server may expose a model under a name that differs from the file on disk. Check the name the server reports, not the file name.

**4.4 A changed config while the app runs.** The app reads the config at start. If the user edits the file while the app runs, the change does nothing until restart. Either say so in the settings menu, or reload on request. Do not reload silently in the middle of an answer.

**4.5 Two copies of one value.** The notes folder path appeared in three places in Course 1. After this lesson, check again with a search. Any copy left behind is a bug waiting to happen.

**4.6 Numbers in JSON are not strings.** A context size written as `"8192"` (with quotes) is a string, and a strict loader rejects it. Decide whether to accept both, and write the decision down.

---

## 5. The lab in outline

You will:

1. write down every hardcoded value (the Exercise 0 table from Lesson 1, updated),
2. define the provider interface and a local implementation,
3. move the endpoints, model names, context size, and timeout into the config,
4. validate every field on load, with a warning for each invalid value,
5. show a check that explains a failed connection,
6. make the server's context size visible to the user, and warn on a mismatch.

The details are in [exercises.md](exercises.md).

---

## 6. Glossary

- **Seam:** a place where one part can be changed without touching the others.
- **Provider:** the object that talks to a model server.
- **Endpoint:** the address the provider sends requests to.
- **Context size:** the number of tokens a model accepts in one request.
- **Timeout:** the longest time a request may take before it fails.
- **Validation:** checking a value against rules before use.
- **Normalise:** convert a value to one standard form, so that comparisons work.

---

## 7. Summary

- The loop talks to an interface. Only the provider knows the server.
- Every server setting lives in the config file, once.
- The context size exists in two places; the app and the server must agree.
- Validation accepts, replaces with a default and warns, or reports. It never crashes.
- Errors name the endpoint, the kind of failure, and the likely fix.

Continue to [exercises.md](exercises.md).
