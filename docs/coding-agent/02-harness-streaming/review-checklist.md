# Module 2, Lesson 2 — Review Checklist

Check each item by doing it, not by believing it.

## Loop and roles

- [ ] I can list the roles in one tool-using turn and say what each role means to the model.
- [ ] I know which message a tool result belongs to, and how its call identifier links them.

## Streaming

- [ ] The answer appears token by token, and time to first token is much shorter than total time for a long answer.
- [ ] The history receives one complete assistant message per answer, never a partial one.
- [ ] A simulated interruption leaves no partial message in the history, and the error is shown.
- [ ] Tool calls that arrive in pieces run only after the stream has ended.

## Display and history

- [ ] The printed history contains no prefix, colour codes, or line decorations.
- [ ] Every `Console.Write` for streamed text is routed through one place, or I have noted why not.

## Working indicator

- [ ] The indicator appears within about one second of pressing Enter.
- [ ] It clears when the first token arrives, and the answer starts on a clean line.
- [ ] A `finally` block (or equivalent) stops it after an exception.

## Cancel

- [ ] Esc stops a long answer within about a second, and the prompt returns.
- [ ] After a cancel, the next question is answered normally.
- [ ] I have written what the history contains after a cancel, and why.

## Timing

- [ ] Each turn prints time to first token and total time.
- [ ] Tool time is measured directly, not calculated by subtraction.
- [ ] No timing line shows a total shorter than the first-token time.

## Edge cases

- [ ] I ran every row of the Exercise 9 table and wrote a result for each.
- [ ] Swedish characters display correctly in the terminal I use.

## Reflection

- [ ] I answered the three Observe questions with a case, not a feeling.
- [ ] I can say which of my changes would be hardest to undo, and why.
