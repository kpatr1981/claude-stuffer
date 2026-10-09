# lean-agents

## What
Cut the fixed per-step cost of every agent. Measured on 2026-10-10 from session transcripts: an agent whose frontmatter has no `tools:` line starts at 30k to 35k tokens of context before reading anything, because it carries every tool schema (all MCP servers) and the skill listings; the scout, with five named tools, starts at 10k. A 91-step builder therefore re-sent about 1.8M tokens of tool catalogue it never used. Three changes:

1. Every bench agent and every generated project role gets a fixed tool list. Writers: `Bash, Read, Edit, Write, Grep, Glob`. Read-only roles keep theirs.
2. Builders are told to keep long command output out of their transcript.
3. The routing block says to start builders early in a session, never near its end.

Version 0.2.3.

## Files
- `claude-stuffer`: `VERSION` (line 25); a new constant next to the protocol constants (lines 169-181); the `senior`, `mid`, `junior`, `qa` bench entries (frontmatter and one added paragraph each); `render_agent` (lines 677-709); the Cost sub-bullet in `RULES`.
- `tests/test_stuffer.py`: the tests qa writes and the ones it must change (`test_init_writes_seven_agents_with_models_and_tools`, `test_agent_frontmatter_has_name_description_level_and_model`, the version test → `test_version_prints_0_2_3`).
- `README.md`: line 96 (the paragraph starting "It writes one file per role") only.
- `/Users/kpatronas/Documents/proj/claude-model-split.md` (outside the repo): one new bullet in `## Keeping the team efficient`, after the **Scouts look up, they do not survey.** bullet. Nothing else.

## Constraints
All text verbatim.

### Script: constants
Add after `REPORT_LINE` (line 176):
```
WRITER_TOOLS = "Bash, Read, Edit, Write, Grep, Glob"
OUTPUT_LINE = ("Keep long output out of your transcript: send test runs, builds and logs to a file and read "
               "back only the tail or the failing part. Everything you print is re-sent on every later step.")
```

### Script: bench frontmatter
In the `senior`, `mid`, `junior` and `qa` entries, add the line `tools: Bash, Read, Edit, Write, Grep, Glob` directly after the `model:` line. `reviewer`, `scout`, `pm` are unchanged.

### Script: bench bodies
Insert this paragraph, as its own paragraph:
```
Keep long output out of your transcript: send test runs, builds and logs to a file and read back only the tail or the failing part. Everything you print is re-sent on every later step.
```
- `senior`, `mid`, `junior`: immediately before the paragraph that starts "If tests fail:".
- `qa`: as the last item of the "Rules:" list, as a bullet: `- Keep long output out of your transcript: send test runs to a file and read back only the tail or the failing part. Everything you print is re-sent on every later step.`

### Script: render_agent
- Frontmatter: replace
```
    if role.get("tools"):
        head.append("tools: " + role["tools"])
```
with
```
    head.append("tools: " + (role.get("tools") or WRITER_TOOLS))
```
- Body: in the `else` branch that appends `FAILURE_PARAGRAPH, REPORT_LINE`, append `OUTPUT_LINE` before them: `body += [OUTPUT_LINE, FAILURE_PARAGRAPH, REPORT_LINE]`. The read-only branch is unchanged.

### Script: RULES Cost sub-bullet
After the sentence ending `because each handoff is a cold start that re-reads the same files.` add, continuing the same bullet at the same indent:
```
    Start a builder early in a session, never near its end: a session that closes mid-run pays for the
    spawn twice. Agents carry only the tools they need; the full tool catalogue is 20k tokens a step.
```

### README line 96
Replace the last sentence `Read-only roles (`security-reviewer`) get a restricted tool list and never modify a file.` with:
`Every role gets a fixed tool list: writers get Bash, Read, Edit, Write, Grep and Glob, and read-only roles (`security-reviewer`) drop Edit and Write and never modify a file. An agent with no tool list carries every tool schema, about 20k tokens, on every step.`

### Article bullet
```
- **Agents carry only the tools they need.** An agent with no tool list starts every step with every tool schema and skill listing in front of it: 30k to 35k tokens before it has read a file, measured against 10k for the scout with five named tools. A 91-step builder re-sent 1.8M tokens of tool catalogue it never touched. Every role the tool writes now has a fixed list, and builders are told to keep test and build output in files, because everything they print is re-sent on every later step.
```

- Keep surrounding text byte-identical. House style: plain sentences, no em-dashes, no emoji.
- Do not run `claude-stuffer init` against the real home. Temp HOME only.
- Article is outside git: copy to the scratchpad first and paste `diff -u` in build.md.

## Done means
- `./claude-stuffer --version` prints `0.2.3`.
- `HOME=<tmp> ./claude-stuffer init`: all seven agent files have a `tools:` line; senior, mid, junior, qa have exactly `Bash, Read, Edit, Write, Grep, Glob`; reviewer, scout, pm unchanged; senior/mid/junior bodies contain "Keep long output out of your transcript".
- Staffing a temp project (the tests' `full_answers`): every writer role file has `tools: Bash, Read, Edit, Write, Grep, Glob` and the output paragraph; `security-reviewer` still has `Read, Grep, Glob, Bash` and no output paragraph.
- Rules block contains "Start a builder early in a session".
- `git diff --stat`: only `claude-stuffer`, `tests/test_stuffer.py`, `README.md`, ticket folder.

## Tests that must pass
`python3 -m unittest discover -s tests -v`: all existing tests (two adjusted as above) plus new ones: writer bench tools exact; project writer role tools exact and OUTPUT paragraph present; security-reviewer unchanged and without the paragraph; rules block has "Start a builder early"; version 0.2.3.

## Notes
- Numbers: artist agent-a5dc533b585943801 (91 steps, first-step context 34,386); scouts 10,104 to 10,574; main session 50,371. From ~/.claude/projects transcripts, 2026-10-10.
- Report to `docs/tickets/lean-agents/build.md`; reply in chat with one line: verdict plus path.
