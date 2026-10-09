# tests: cost-rule

Verdict: tests written, 3 fail for the right reason, 87 others pass (90 total).

File changed: tests/test_stuffer.py only.

- TestInit.test_init_rules_block_contains_cost_rule_and_compact: init into temp HOME; rules block contains "- Cost:" and "`/compact`".
- TestInit.test_init_rules_cost_bullet_after_parallel_work_before_handoffs: in the block, index("are always sequential.") < index("- Cost:") < index("- Handoffs carry context").
- TestCli.test_version_prints_0_2_1 (renamed from _0_2_0): `--version` output contains "0.2.1".
- Helper added: TestInit._rules_block().

Current failures (python3 -m unittest discover -s tests -v):
- test_version_prints_0_2_1: AssertionError: '0.2.1' not found in 'claude-stuffer 0.2.0\n'
- test_init_rules_block_contains_cost_rule_and_compact: AssertionError: '- Cost:' not found in <rules block, no Cost bullet>
- test_init_rules_cost_bullet_after_parallel_work_before_handoffs: AssertionError: -1 == -1 : no Cost bullet in rules block

Ran 90 tests, FAILED (failures=3).
