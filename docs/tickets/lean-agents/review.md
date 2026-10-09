PASS

# review.md (reviewer) - lean-agents

## 1. Spec coverage and scope
- `git diff --stat` shows README.md (2), claude-stuffer (23), tests/test_stuffer.py (36), and docs/tickets/lean-agents/ is untracked. Nothing else changed.
- claude-stuffer:25 has `VERSION = "0.2.3"`. `./claude-stuffer --version` prints `claude-stuffer 0.2.3`.
- claude-stuffer:177-179: WRITER_TOOLS and OUTPUT_LINE sit right after REPORT_LINE and match the spec exactly. The OUTPUT_LINE string is split at "read " / "back", so the joined text is identical to the spec.
- Bench frontmatter: `tools: Bash, Read, Edit, Write, Grep, Glob` comes straight after `model:` in senior, mid, junior and qa. The diff does not touch the reviewer, scout or pm entries.
- Bench bodies: in senior, mid and junior the output paragraph is its own paragraph, directly before "If tests fail:". In qa it is the last "Rules:" bullet, using the shorter "send test runs to a file" wording, as the spec says.
- render_agent (claude-stuffer:702): `head.append("tools: " + (role.get("tools") or WRITER_TOOLS))`. claude-stuffer:730: `body += [OUTPUT_LINE, FAILURE_PARAGRAPH, REPORT_LINE]` is only in the writer `else` branch. The read-only branch is unchanged. Both match the spec.
- RULES Cost sub-bullet: the two lines follow "...re-reads the same files." and keep the same 4-space indent. The text matches the spec.
- README.md:96: only the last sentence was replaced, and the new text matches the spec. No other README lines changed.
- Article (/Users/kpatronas/Documents/proj/claude-model-split.md): `diff -u` against the builder's copy (scratchpad/before.md) shows one added line. It is the spec bullet, placed right after the "Scouts look up, they do not survey." bullet. Nothing else differs. scratchpad/article.before.md is an older copy that predates the earlier scout edits, so it is not the baseline for this ticket.
- Nothing was added beyond the spec.

## 2. Tests
- `python3 -m unittest discover -s tests -v`: Ran 95 tests, OK. I ran the full set myself, not a subset.
- The test changes match tests.md, plus one change from the architect note: BENCH_TOOLS["reviewer"] = "Read, Grep, Glob, Bash, Write" (tests/test_stuffer.py:521). That is the reviewer's real, unchanged tool list, so the fix is correct. The test does not weaken the spec.

## 3. Manual checks
- `HOME=<tmp> ./claude-stuffer init` exited 0:
  - senior, mid, junior and qa have `tools: Bash, Read, Edit, Write, Grep, Glob` and the output paragraph.
  - reviewer has `Read, Grep, Glob, Bash, Write`, scout and pm have `Read, Grep, Glob, Write, Edit`, and none of the three has the paragraph.
  - The CLAUDE.md rules block contains "Start a builder early in a session".
- I staffed a temp project (`--project <tmp> --answers <json>`) with the tests' full_answers plus `addons: ["security-reviewer"]`. It exited 0:
  - cli-developer and ksh-engineer: `tools: Bash, Read, Edit, Write, Grep, Glob` and the output paragraph, just before the failure line.
  - security-reviewer: `tools: Read, Grep, Glob, Bash` and no output paragraph.

## 4. Safety and style
- No secrets, no new input handling and no destructive operations.
- The added lines contain no em-dashes or en-dashes, no emoji, and only plain sentences.
- The new `tools:` lines are valid YAML plain scalars, and the frontmatter structure is unchanged.
- Non-blocking, pre-existing, not part of this change: some generated project-role descriptions contain ": " (for example "Builds the command-line tool: flags, ..." in cli-developer.md). Strict YAML rejects that in a plain scalar. Claude Code parses it fine, and this ticket does not touch it. A later ticket could quote descriptions in render_agent if strict YAML ever matters.

## 5. Would I approve this as a PR?
Yes. The change is small and exact, and its tests cover the frontmatter, the body paragraph, the read-only exclusion, the rules text and the version.
