# Module 6, Lesson 18 — Review Checklist

## Hook file

- [ ] Hooks are read from one file the user writes, and validated on load.
- [ ] An unknown event, a program with a path, or an out-of-range timeout is refused with a warning.

## Running

- [ ] Each hook runs with an argument list, the project root as working directory, a clean environment, a timeout, and an output limit.
- [ ] The `{file}` placeholder is one argument, and a file name with a space is tested.
- [ ] A timeout stops the whole process tree, and the result is a fail with a note.

## Results

- [ ] Each hook's result appears in the turn, with a pass or fail mark and a duration.
- [ ] A failing hook shows the last lines of its output.
- [ ] I decided, and wrote down, whether the model sees each hook's output.

## Protection

- [ ] The model cannot write the hook file, in any mode, and the rule is tested in all three.
- [ ] Hook output is shown as data and is not followed.

## Visibility

- [ ] `/hooks` lists each hook, its event, its match, its command, and whether the model sees its output.
- [ ] Each run is logged in the decision log, without the output.

## Blocking

- [ ] A blocking hook stops the action, and the model receives a result that names the hook.
- [ ] I documented where a blocking hook sits relative to the approval prompt.

## Reflection

- [ ] I measured how often my prompt-only check was skipped, and how the hook changed that.
- [ ] I can say why the model must not be able to change the hook file.
