# Module 2, Lesson 6 — Review Checklist

## Engine

- [ ] The evaluation order is deny, then allow, then ask, then the default of ask, and it is written down.
- [ ] A deny always wins over an allow, and a test proves it.
- [ ] The engine receives the resolved target, not the model's text.
- [ ] The engine does not contain mode names.

## Modes

- [ ] The three modes are data, and the active mode loads from the config with a fallback to ask.
- [ ] Read-only mode denies a write, and the reason names the mode.

## Single check point

- [ ] Every tool call goes through one check before it is invoked.
- [ ] A denied call returns a tool result, so the conversation stays valid.

## Ask prompt

- [ ] The prompt shows the tool in plain words, the full target, and the choices.
- [ ] Only the exact answers are accepted; anything else is asked again.

## Always

- [ ] "Always" creates a rule with the narrowest scope that makes sense.
- [ ] `/permissions` lists the rules, and removing one restores the ask.
- [ ] A session rule never overrides a mode's deny.

## Log

- [ ] Every decision adds one line to the log.
- [ ] The log contains no file contents.

## Reflection

- [ ] I can explain why the default is ask and not deny, and when deny would be better.
- [ ] I can name one change to the design that reduces approval fatigue.
