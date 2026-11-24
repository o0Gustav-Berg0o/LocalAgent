# Module 4, Lesson 10 — Review Checklist

## Measurement

- [ ] The meter counts the five parts separately, and each message is counted once.
- [ ] I measured a request with no tools, with three tools, and after a file read, and recorded the characters.
- [ ] I checked whether my server reports input token counts, and wrote down the answer.

## Estimate and calibration

- [ ] The estimate uses one named constant, and the README's caveats are in the code comment.
- [ ] If counts are reported, the calibrator uses a rolling average and ignores outliers.
- [ ] The correction factor is stored per model.

## Budget and display

- [ ] The budget is the context size minus the reserve, and the reserve is a setting.
- [ ] The status line shows the percentage, the used tokens, the budget, and the reserve.
- [ ] The status line updates after each answer, not during streaming.

## Thresholds

- [ ] Warn and act thresholds are validated, and the warn value is lower than the act value.
- [ ] Each threshold message prints once when the level changes.

## Refusal

- [ ] A request that would exceed the budget is refused before it is sent, and the server is not called.
- [ ] A refused request is not added to the history.

## Reflection

- [ ] I can name the part of the request that grew most in my session, with a number.
- [ ] I can say how wrong my raw estimate was, for my model and my text.
