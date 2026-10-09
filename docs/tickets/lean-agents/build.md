
# build.md (mid) - lean-agents

Verdict: BLOCKED on one test. Implementation done per spec; 94 of 95 pass.

## Failure
test_init_writes_seven_agents_with_models_and_tools (tests/test_stuffer.py:539):
AssertionError: 'Read, Grep, Glob, Bash, Write' != 'Read, Grep, Glob, Bash' : reviewer

Cause: the test's BENCH_TOOLS expects reviewer = "Read, Grep, Glob, Bash". The script's reviewer entry (unchanged, as the spec requires) has "Read, Grep, Glob, Bash, Write" (matches the live reviewer agent's tool list). The test expectation is wrong, not the script. Fixing it needs either editing tests/test_stuffer.py (forbidden to me) or changing the reviewer entry (spec says unchanged). Stopped, no workaround attempted.

## Changed
claude-stuffer: VERSION 0.2.3; WRITER_TOOLS and OUTPUT_LINE constants; tools line on senior/mid/junior/qa; output paragraph in senior/mid/junior (before "If tests fail:") and qa (last Rules bullet); render_agent tools and body; RULES Cost sub-bullet.
README.md: line 96 only.
/Users/kpatronas/Documents/proj/claude-model-split.md: one bullet (diff shows only that line added; before copy in scratchpad before.md).

## Output
Tests: Ran 95 tests, FAILED (failures=1) (the one above).
Version: claude-stuffer 0.2.3
git diff --stat: README.md (2), claude-stuffer (23), tests/test_stuffer.py (36, qa's earlier change) plus untracked docs/tickets/lean-agents/.
Article diff: single added line after the "Scouts look up" bullet ("Agents carry only the tools they need...").
Fix needed: change reviewer in BENCH_TOOLS (test file) to "Read, Grep, Glob, Bash, Write".

## Architect note (2026-10-10 01:57)
The failing expectation was the architect's error in the qa handoff (reviewer tools listed as four, it has five). Fixed the one string in tests/test_stuffer.py BENCH_TOOLS. 95 tests OK.
