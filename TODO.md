# Course TODO

Ideas and gaps discovered while building the reference implementation, to turn into
guide material (a new lesson, a capstone extension, or a note added to an existing
lesson) once the core 12 lessons are solid. Each entry notes what prompted it, not
just what it is, so the reasoning isn't lost by the time we act on it.

- **"Robustness & Scaling" addendum — timeout handling + context compaction.**
  Two related gaps that both showed up after Lesson 6 and both cost an extra round
  trip on slow local hardware, so they belong in one addendum rather than two
  separate bolt-ons:

  - *Timeout / retry handling for slow or failed model calls.*
    Prompted by: on weaker hardware (e.g. an integrated GPU with no dedicated VRAM),
    a single generation can run past .NET's default ~100s HTTP client timeout, which
    silently cancels the request mid-generation (visible server-side as `cancel task`
    in the `llama-server` log) rather than failing cleanly. `AgentLoop` currently has
    no handling for this distinct from any other tool/model exception — it would just
    throw. Worth covering: configuring a longer client timeout
    (`ClientPipelineOptions.NetworkTimeout`, confirmed via the real exception message
    — see https://aka.ms/net/scm/configure/networktimeout), deciding what "the model
    didn't respond in time" should look like to the user (retry? give up with a clear
    message? exponential backoff for transient network issues specifically, separate
    from "this is just a slow local model"?).
  - *Context compaction when history gets large.*
    Prompted by: Lesson 6 already covers message-count trimming and placeholdering
    old tool results, but neither actually preserves the *content* of what got
    dropped — a summarization step (condense the oldest N turns into one message via
    an LLM call, keep recent turns verbatim) is the natural next step once trimming
    alone starts losing context the model actually needed. Worth flagging explicitly:
    compaction itself is an extra LLM call, on the same slow local hardware the
    timeout item above is about — so this isn't free, and a synchronous
    "compact once a token budget is crossed" trigger probably fits the course's style
    better than a background/proactive one.

  Whether this becomes a Lesson 1/6 addendum or its own short lesson near the end of
  the course is still open.

- **Model-routing gateway (local small model ↔ frontier model), shared retrieval.**
  Prompted by: a "frugal power user" workflow — send cheap/simple turns ("hello") to
  the local Qwen model, and route genuinely heavy tasks ("go through this whole
  codebase") to a frontier model, without re-architecting the agent. The useful
  insight from discussing this: it's actually two separate problems, not one —
  (1) *routing*, which is a natural extension of the existing tool-calling loop (give
  the local model an `escalate_to_frontier(reason)` tool rather than building a
  separate classifier), and (2) *"sharing context,"* which isn't really a context
  problem at all — both models should share the same `search_notes`-style retrieval
  tool over the same index, rather than ever stuffing a whole codebase into either
  model's context window.
  **Explicitly breaks the course's "local GPU only, no cloud fallback" decision** —
  this needs an API key, a network call, cost, and sends code to a third party, so if
  it's built at all it should be framed as an optional, opt-in extension with that
  tradeoff stated up front, not folded into the default course path. Likely sized as
  its own mini-project/capstone extension rather than a short addendum, and might end
  up living outside this course entirely.
