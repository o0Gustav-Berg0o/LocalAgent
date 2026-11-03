# Course TODO

Ideas and gaps discovered while building the reference implementation, to turn into
guide material (a new lesson, a capstone extension, or a note added to an existing
lesson) once the core 12 lessons are solid. Each entry notes what prompted it, not
just what it is, so the reasoning isn't lost by the time we act on it.

- **Timeout / retry handling for slow or failed model calls.**
  Prompted by: on weaker hardware (e.g. an integrated GPU with no dedicated VRAM),
  a single generation can run past .NET's default ~100s HTTP client timeout, which
  silently cancels the request mid-generation (visible server-side as `cancel task`
  in the `llama-server` log) rather than failing cleanly. `AgentLoop` currently has
  no handling for this distinct from any other tool/model exception — it would just
  throw. Worth covering: configuring a longer client timeout, deciding what "the
  model didn't respond in time" should look like to the user (retry? give up with a
  clear message? exponential backoff for transient network issues specifically,
  separate from "this is just a slow local model"?), and whether that's a Lesson 1
  addendum or its own short lesson near the end of the course.
