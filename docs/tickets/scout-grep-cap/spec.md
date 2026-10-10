# scout-grep-cap

## What
drop64 scouts reached ~97k context because of uncapped greps with multi-term alternations (10-18k chars per result) and one 41k read of `.git/logs/HEAD`. Add two rules to the scout bench prompt: narrow greps, no `.git/` reads.

## Files
- `claude-stuffer` (scout prompt, the "Budget:" paragraph near line 447; the version string)
- `README.md` (scout row, line 26)
- `tests/test_stuffer.py` (one new test next to `test_init_scout_bench_file_has_30_call_budget_and_not_reached`, ~line 574)

## Constraints
- In `claude-stuffer`, append exactly this text to the end of the "Budget:" paragraph (same line, one space before it):
  `Grep narrow: list matching files first (output_mode files_with_matches), then show matching lines only for the narrowed path with head_limit 50; one term per search, never an alternation of several. Never read anything under .git/; git history is a "Not reached" item for the builder, who has Bash.`
- In `README.md` scout row, change `at most 30 tool calls, writes` to `at most 30 tool calls, narrow greps (files first, at most 50 lines), never reads `.git/`, writes` (keep the backticks around .git/).
- New test `test_init_scout_bench_file_caps_greps_and_skips_git`: same setup as the neighbouring test; assert `"head_limit 50"` and `"Never read anything under .git/"` are in scout.md.
- Bump the patch version (0.2.6 -> 0.2.7).
- Change nothing else.

## Done means
- The three edits above, the version bump, full suite green, one commit on main: `Scout: narrow greps, no .git reads` with the attribution line `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`. Do not push. Do not touch `~/.claude`.
- `build.md` in this folder: what changed, the last 5 lines of test output, commit hash.

## Tests that must pass
- `python3 -m unittest discover -s tests` (all green, one more test than before)

## Notes
Evidence: drop64 session 2a0641de, scouts a14c/a6a7/ab61.
