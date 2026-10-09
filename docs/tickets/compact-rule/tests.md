# tests: compact-rule

File: tests/test_stuffer.py. New class `CompactRule` (16 tests) plus the version test renamed to
`test_version_prints_0_2_6` (expects 0.2.6). Existing 96 tests unchanged; the Cost-bullet wording
tests at 586-600 still pass as written (they check "- Cost:" and "`/compact`").

Tests (all use the temp home via run()/init):
- rule: ticket-closes trigger ("ticket closes" + `/compact`), "150k" trigger, "never poll a running
  builder" + "completion notice", Cost bullet text and position kept.
- init: writes ~/.claude/hooks/context-guard.py; --dry-run mentions context-guard and writes nothing;
  stdout has the UserPromptSubmit snippet naming context-guard.py; settings.json never created, and an
  existing one is byte-identical after `init --force`.
- hook (runs the written script with python, stdin JSON): 200k -> one line with "200k", "150k", "/compact";
  50k -> empty stdout; env CLAUDE_CONTEXT_GUARD=10000 makes 50k print; =500000 makes 200k silent;
  missing transcript, non-JSON stdin, garbage transcript -> empty stdout, exit 0.
- `test_hook_uses_last_assistant_message_only` is weak (passes once the hook exists, only checks 50k
  silent with a tiny earlier message); the senior may tighten it.

Contract assumed: usage sum = input + cache_read + cache_creation of the last `type: assistant` line;
output formats the token count as "<N>k" (200000 -> "200k"); reminder contains "150k" and "/compact".

Result: 112 run, 13 failures, 0 errors. Failing for the right reason (feature missing):
- 6 hook tests + hook-file test: script absent, `python hooks/context-guard.py` returns exit 2 / no file.
- init dry-run/snippet: stdout lacks "context-guard" / "UserPromptSubmit".
- rule: "150k" and "never poll a running builder" not in the rules block.
- version: still 0.2.5.
Passing already (by design, guard current behaviour): ticket-closes wording, Cost position, settings.json
untouched, bad-input hook tests are among the failures only because the script does not exist.
Full output: /private/tmp/claude-501/qa.txt
