# architect-opus

## What
The architect (main session) moves from Fable 5.1 to Opus 5.5. The user set `"model": "claude-opus-5-5"` in ~/.claude/settings.json on 2026-10-10. Update every text that names the architect's model. Version 0.2.5. Mechanical string edits only.

## Files
- `claude-stuffer`: line 4 (docstring), `INIT_HINT` (lines 185-186), RULES lines 376 and 434, `VERSION`.
- `tests/test_stuffer.py`: version test → `test_version_prints_0_2_5`, assert `0.2.5`.
- `README.md`: lines 15 and 77.
- `/Users/kpatronas/Documents/proj/claude-model-split.md`: lines 13 and 160.

## Constraints (verbatim)
- Script line 4, RULES 376 and 434: replace `architect (Fable)` with `architect (Opus)`.
- `INIT_HINT`: replace both `claude-fable-5-1` with `claude-opus-5-5`.
- README line 15: replace `| Fable (the main session) |` with `| Opus (the main session) |`.
- README line 77: replace both `claude-fable-5-1` with `claude-opus-5-5`.
- Article line 13: replace `| Fable 5.1 |` with `| Opus 5.5 |`.
- Article line 160: replace the sentence pair `Fable wrote one spec and read three short reports. Opus did the building and the two reviews.` with `Opus wrote one spec and read three short reports, did the building and the two reviews.`
- Nothing else changes. `grep -i fable` over claude-stuffer, README.md and the article must then return nothing.

## Done means
- `./claude-stuffer --version` prints `0.2.5`.
- `grep -ci fable claude-stuffer README.md /Users/kpatronas/Documents/proj/claude-model-split.md` prints 0 for each.
- `python3 -m unittest discover -s tests 2>&1 | tail -3` is OK (96 tests).
- `git diff --stat` shows only `claude-stuffer`, `tests/test_stuffer.py`, `README.md` plus the ticket folder.

## Tests that must pass
The full suite, all green, with the renamed version test.

## Notes
- Report to `docs/tickets/architect-opus/build.md`; reply in chat with one line: verdict plus path.
