# Build: scout-grep-cap (junior, direct run)

## What changed
- `claude-stuffer`: appended the "Grep narrow" rule to the end of the scout "Budget:" paragraph (line 447); VERSION 0.2.6 -> 0.2.7.
- `README.md`: scout row now reads "narrow greps (files first, at most 50 lines), never reads `.git/`".
- `tests/test_stuffer.py`: version assertion 0.2.6 -> 0.2.7; new test `test_init_scout_bench_file_caps_greps_and_skips_git` asserting "head_limit 50" and "Never read anything under .git/" in scout.md.
- `docs/tickets/scout-grep-cap/spec.md` and `build.md` committed with the change.

## Test output (tail)
```
....................................................................................................................
----------------------------------------------------------------------
Ran 116 tests in 7.440s

OK
```
