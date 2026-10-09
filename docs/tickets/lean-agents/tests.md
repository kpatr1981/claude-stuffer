# tests.md (qa) - lean-agents

Verdict: tests written, 6 fail for the right reason, 89 others pass (95 total). Only tests/test_stuffer.py was edited.

## Changed
- TestCli.test_version_prints_0_2_3 (renamed): asserts "0.2.3". Fails: output is 'claude-stuffer 0.2.2'.
- TestInit.test_init_writes_seven_agents_with_models_and_tools: every bench file has `tools`; senior/mid/junior/qa == "Bash, Read, Edit, Write, Grep, Glob"; reviewer "Read, Grep, Glob, Bash"; scout and pm "Read, Grep, Glob, Write, Edit" (class constants BENCH_TOOLS, OUTPUT). Fails: no tools key for senior.
- TestGeneration.test_agent_frontmatter_has_name_description_level_and_model: project writer role tools == "Bash, Read, Edit, Write, Grep, Glob" (was assertNotIn). Fails: KeyError 'tools'.
- TestVariants.test_security_reviewer_addon_is_read_only: added assertNotIn of the output paragraph (tools check already existed). Passes now; guards against regression.

## New
- TestInit.test_init_senior_mid_junior_qa_bodies_have_output_paragraph: each of the four bench files contains "Keep long output out of your transcript". Fails: not found in senior.md.
- TestGeneration.test_project_writer_role_has_output_paragraph: staffed cli-developer and custom ksh-engineer files contain the paragraph. Fails: not found.
- TestInit.test_init_rules_block_says_start_builder_early: rules block contains "Start a builder early in a session". Fails: not found.

## Failure output (python3 -m unittest discover -s tests -v)
Ran 95 tests; FAILED (failures=5, errors=1)
- test_version_prints_0_2_3: AssertionError '0.2.3' not found in 'claude-stuffer 0.2.2\n'
- test_agent_frontmatter_has_name_description_level_and_model: KeyError: 'tools'
- test_project_writer_role_has_output_paragraph: 'Keep long output out of your transcript' not found in cli-developer file
- test_init_rules_block_says_start_builder_early: 'Start a builder early in a session' not found in rules block
- test_init_senior_mid_junior_qa_bodies_have_output_paragraph: paragraph not found in senior.md
- test_init_writes_seven_agents_with_models_and_tools: 'tools' not found in frontmatter of senior
