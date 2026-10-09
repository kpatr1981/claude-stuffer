# scout-lookup tests

Verdict: 3 tests fail for the right reason (feature missing); the other 89 pass (92 total).

File changed: tests/test_stuffer.py only.

- TestCli.test_version_prints_0_2_2 (renamed from _0_2_1): `--version` output contains "0.2.2".
  Fails: AssertionError: '0.2.2' not found in 'claude-stuffer 0.2.1\n'
- TestInit.test_init_scout_bench_file_has_30_call_budget_and_not_reached: after init into temp HOME, ~/.claude/agents/scout.md contains "at most 30 tool calls" and "Not reached".
  Fails: 'at most 30 tool calls' not found in the old scout text ("Answer the question you are asked...").
- TestInit.test_init_rules_block_says_lookups_not_surveys: rules block in ~/.claude/CLAUDE.md contains "Lookups, not" and `"how does X work" goes to the builder`.
  Fails: 'Lookups, not' not found in the old rules block (old "one scout per ticket with three or four questions" text).

Run: `python3 -m unittest discover -s tests -v` -> Ran 92 tests, FAILED (failures=3).
Not tested (spec lists only these three): "30 tool calls at" in the rules block, README, article.
