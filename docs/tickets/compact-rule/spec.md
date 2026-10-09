# compact-rule

## What
The `/compact` advice in the Cost bullet is not followed: a measurement of 2026-10-09/10 showed the
main session at 59% of all spend (86% counting one 963k-context session), almost all of it cache
reads on long transcripts, against ~24% that a Sonnet architect would save. Make the rule concrete
and give it an optional enforcer: compact after every closed ticket or once the context passes
150k, and never poll a running builder (background agents notify when they finish). Ship a small
`UserPromptSubmit` hook that reminds the architect when its context is past the threshold.

## Files
- `claude-stuffer` (routing block text in the Cost bullet; new hook script text embedded like the
  bench; `init` writes it; `VERSION` bump to 0.2.6)
- `tests/test_stuffer.py`
- `README.md`
- `~/Documents/proj/claude-model-split.md` (article; the "Compact between tickets" bullet, ~line 249,
  and the cost paragraph ~line 115)
- `docs/tickets/compact-rule/build.md` (report)

## Constraints
- Stdlib only, Python 3.9+, one-file script, as today.
- Rule text: replace "`/compact` once a ticket closes ..." with a version that names both triggers
  (ticket closed, or context past 150k) and adds "never poll a running builder: wait for its
  completion notice". Keep the rest of the Cost bullet and its position (tests at
  tests/test_stuffer.py:586-600 must still pass, adjusted only where the wording changed).
- Hook: `init` writes `~/.claude/hooks/context-guard.py` (honour the same home override the tests
  already use for `init`). The script reads the hook JSON on stdin, opens `transcript_path`, takes
  the last assistant message's usage (input + cache_read + cache_creation tokens), and if it is
  over the threshold prints one line to stdout, e.g.
  `Context is 183k tokens (> 150k). Finish or close the current ticket, then /compact.`
  Under the threshold, or on any error (missing file, bad JSON), print nothing and exit 0: the hook
  must never block a prompt. Threshold 150000, overridable by env `CLAUDE_CONTEXT_GUARD`.
- `init` must NOT edit `settings.json`. It prints the snippet to add under `hooks.UserPromptSubmit`
  and says so; README shows the same snippet. Verify the hook JSON shape and the stdout-becomes-
  context behaviour against the Claude Code hooks docs and note the source in build.md.
- Do not touch the bench agent files or other rules.

## Done means
- `claude-stuffer init --dry-run` (or the existing preview path) shows the new Cost wording and the
  hook file; a real `init` into a temp home writes `hooks/context-guard.py` and prints the snippet.
- Feeding the hook a fake transcript with 200k usage prints the reminder; 50k prints nothing;
  a missing transcript prints nothing; all exit 0.
- README and article describe the two triggers, the no-polling rule and the optional hook.
- `--version` prints 0.2.6.

## Tests that must pass
- `python3 -m unittest discover -s tests -v` (all existing 96 plus new ones for: rule wording,
  hook file written by init, hook over/under/error cases, settings.json untouched).

## Notes
- Measurement script: `/private/tmp/claude-501/-Users-kpatronas/8341b1d2-7fc6-4db8-9706-71c17d7c8534/scratchpad/share.py`
  (per-role share across session jsonl files). Usage fields live in `message.usage` of
  `type: assistant` lines.
- Not in scope: applying the new block to `~/.claude/CLAUDE.md`, installing the hook in the user's
  settings, Ghost republish, push. Those follow after the reviewer PASS.
- Route: qa (Sonnet) → senior (Opus; judgement in the hook contract) → reviewer.
