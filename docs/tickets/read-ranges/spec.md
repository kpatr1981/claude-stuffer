# read-ranges

## What
Tonight's E4 builder read 174k tokens of source over 45 steps before its first write, mostly whole assembly files. The "keep long output out of your transcript" paragraph added in 0.2.3 covers command output but not file reads, which have the same effect. Add one sentence to that paragraph everywhere it appears. Version 0.2.4.

## Files
- `claude-stuffer`: `VERSION`; the `OUTPUT_LINE` constant; the output paragraph in the `senior`, `mid`, `junior` bench bodies and the qa Rules bullet.
- `tests/test_stuffer.py`: the version test (`test_version_prints_0_2_3` → `test_version_prints_0_2_4`, assert `0.2.4`); one new assertion.

## Constraints
- Everywhere the sentence `Everything you print is re-sent on every later step.` appears (the constant and the four bench texts), append after it, same paragraph: ` Read the lines you need, not whole files: a file read once is re-sent on every later step too.`
- In the constant `OUTPUT_LINE`, keep the Python string concatenation style of the neighbouring constants.
- New test, next to the existing output-paragraph tests: after `init` into a temp HOME, `senior.md` contains `Read the lines you need, not whole files`; and a staffed project writer role file contains it too.
- Nothing else changes. No README or article change.

## Done means
- `./claude-stuffer --version` prints `0.2.4`.
- `grep -c "Read the lines you need, not whole files" claude-stuffer` prints 5.
- `python3 -m unittest discover -s tests -v` is all OK (96 tests).
- `git diff --stat` shows only `claude-stuffer` and `tests/test_stuffer.py` plus the ticket folder.

## Tests that must pass
`python3 -m unittest discover -s tests -v`, all green, including the renamed version test and the new one.

## Notes
- Report to `docs/tickets/read-ranges/build.md`; reply in chat with one line: verdict plus path.
