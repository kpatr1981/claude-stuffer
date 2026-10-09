# scout-lookup

## What
Redefine the scout from a surveyor into a lookup tool. Evidence from 2026-10-10 in the drop64/game_engine project: the E3 scout ran 214 steps to a 300k transcript (34M cached input tokens re-read) and wrote a 507-line scout.md; the builder then read all the source anyway. The E4 scout, given four "how does X work" questions, passed 144 steps and 200k context with nothing written yet. A scout that surveys is the most expensive way to read code, because every tool call re-sends everything it has read, and the result is paid for twice.

The new scout: answers lookups (which files a change touches, which tests cover them, where a symbol is used), refuses surveys ("how does X work" is the builder's question), has a hard budget of 30 tool calls, writes scout.md as it goes, and ends with a "Not reached" list when the budget runs out. The routing block, README and article say the same. Version 0.2.2.

## Files
- `claude-stuffer`: `VERSION` (line 25); the `"scout"` entry of the bench dict (lines 323-334 at commit 8d25d91); in `RULES`, the Cost sub-bullet's "So:" sentence (lines 383-384) and the `scout` role bullet (lines 400-401).
- `tests/test_stuffer.py`: the version test (`test_version_prints_0_2_1` → `test_version_prints_0_2_2`, assert `0.2.2`), plus the new tests qa writes.
- `README.md`: the `scout` row of the roles table (line 26) only.
- `/Users/kpatronas/Documents/proj/claude-model-split.md` (outside the repo): the Scout table row (line 35), the sentence at line 175 and the **Narrow scout questions** bullet (line 247). Nothing else.

## Constraints
All replacements verbatim.

### Script: the scout bench entry
Replace the whole `"scout": """\ ... """,` value with:

```
---
name: scout
description: Scout (Haiku, junior level, no code edits). Looks things up in the codebase for the architect before a spec is written: which files a change touches, which tests cover them, where a symbol is used. Lookups, not surveys; at most 30 tool calls.
model: haiku
tools: Read, Grep, Glob, Write, Edit
---
You are the scout. You answer lookup questions about the codebase: where something lives, what calls it, which tests cover it. You do not survey a subsystem or explain how it works; if a question needs that, say so in one line and leave it to the builder, who reads the code anyway.

Budget: at most 30 tool calls in total, fewer if the handoff says so. Read docs/tickets/FACTS.md first if it exists and do not re-derive what it already says. Prefer Grep and Glob to Read; read only the lines you need, never a whole file to find one symbol.

Write as you go: after each question is answered, append it to scout.md in the ticket folder named in the handoff, so a cut-off run still leaves its findings. When the budget is spent, stop, add a final section "Not reached" listing the questions you did not answer and the best place to look for each, and report. Append any durable fact (where something lives, which tests cover what) as one line to docs/tickets/FACTS.md. Reply in chat with one line: done or partial, and the path. Those two files are the only things you ever write.

Report facts with file paths and line numbers. Quote the relevant lines rather than paraphrasing them. Do not guess.
```

### Script: RULES, Cost sub-bullet
Replace these two lines:
```
    So: one scout per ticket with three or four questions, not one broad sweep; `/compact` once
    a ticket closes so the next task starts from the ticket folder, not the conversation; Opus
```
with:
```
    So: scouts do lookups, not surveys (one question or a few on the same files, 30 tool calls at
    most, "how does X work" goes to the builder); `/compact` once
    a ticket closes so the next task starts from the ticket folder, not the conversation; Opus
```

### Script: RULES, scout role bullet
Replace:
```
  - `scout` (Haiku, no code edits, writes only scout.md and FACTS.md): answers codebase questions before a spec is written.
    Use it instead of reading files in the main session.
```
with:
```
  - `scout` (Haiku, no code edits, writes only scout.md and FACTS.md): looks things up before a spec is
    written: which files a change touches, which tests cover them, where a symbol is used. Lookups, not
    surveys: one question or a few on the same files, at most 30 tool calls, writes scout.md as it goes
    and ends with a "Not reached" list. "How does X work" is the builder's question, not the scout's.
    Use it instead of reading files in the main session.
```

### README, roles table row
Replace line 26 with:
```
| `scout` | Junior, no code edits | Looks things up for the architect: which files a change touches, which tests cover them, where a symbol is used. Lookups, not surveys: at most 30 tool calls, writes `scout.md` as it goes. "How does X work" goes to the builder. |
```

### Article
Line 35, replace with:
```
| Scout | Junior | Haiku, no code edits | Looks things up for the architect: which files a change touches, which tests cover them, where a symbol is used. Facts with file paths and line numbers, at most 30 tool calls, written as it goes. Not a surveyor: "how does X work" is the builder's question. |
```
Line 175, replace `The architect asks the scout what it needs to know, then writes a spec.` with `The architect asks the scout for the lookups the spec needs (files, tests, callers), then writes it.` Keep the rest of the line.

Line 247, replace the whole **Narrow scout questions** bullet with:
```
- **Scouts look up, they do not survey.** The first version of this setup let the scout answer "how does the camp state machine work". One such scout ran 214 steps to a 300k-token transcript, re-reading 34 million cached tokens on the way, and the builder then read the same source itself. A scout's transcript grows with everything it reads and is re-sent on every step, so a survey is the most expensive way to read code. Now the scout gets lookups only (which files, which tests, who calls this), a budget of 30 tool calls, and writes its notes as it goes so a cut-off run still leaves something. "How does X work" goes to the builder, who reads the code anyway.
```

- Keep all surrounding text byte-identical. House style: plain sentences, no em-dashes, no emoji.
- Do not run `claude-stuffer init` against the real home. The architect does that after review.
- The article folder is not a git repo: copy the file to the scratchpad first and paste `diff -u` into build.md.

## Done means
- `./claude-stuffer --version` prints `0.2.2`.
- `HOME=<tmpdir> ./claude-stuffer init` writes `~/.claude/agents/scout.md` containing "at most 30 tool calls" and "Not reached", and a CLAUDE.md rules block containing "Lookups, not" and "30 tool calls at".
- `git diff --stat` shows only `claude-stuffer`, `tests/test_stuffer.py`, `README.md` and the ticket folder.
- The article diff shows only the three edits above.

## Tests that must pass
- `python3 -m unittest discover -s tests -v`: the existing 90 plus the new ones from `tests.md`: scout bench file contains "at most 30 tool calls" and "Not reached"; rules block contains "Lookups, not" and `"how does X work" goes to the builder`; `--version` prints `0.2.2`.

## Notes
- Measured 2026-10-10 01:40 from the session transcripts under ~/.claude/projects; numbers quoted in the article are those.
- Report to `docs/tickets/scout-lookup/build.md`; reply in chat with one line: verdict plus path.
