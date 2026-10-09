# cost-rule

## What
Add a cost rule to the team-by-seniority setup, in three places: the routing block `claude-stuffer init` writes (the `RULES` string in the script), the README paragraph that summarises that block, and the article. The rule, distilled from a session on 2026-10-10 where the usage panel was read:

- A subagent costs the same whether it runs alone or beside another. Parallel saves wall-clock time, not tokens.
- What drives cost: the number of spawns (each starts cold and re-derives context), the model each one runs on (Opus roles are where the tokens go), and the length of every transcript, because each tool call re-sends the whole transcript so far. A scout that answered eleven questions over 147 tool calls used about 300k tokens; that is the same whether or not a junior ran next to it. The main session is a transcript too: 77% of that day's usage was turns at over 150k context.
- The agents do not pay for talking to each other. A handoff is one prompt in and one line back; reports go through files. The cost is reading, not communicating.
- Therefore: one scout per ticket with three or four questions, not one broad sweep; `/compact` once a ticket closes so the next task starts from the ticket folder, not the conversation; Opus only where judgement is needed.

Bump the version to 0.2.1 because the embedded rules text changes what `init` writes.

## Files
- `claude-stuffer`: the `RULES` string (lines 357-412 at commit ee28a04) and `VERSION` at line 25.
- `tests/test_stuffer.py`: the version test at lines 255-258 (rename to `test_version_prints_0_2_1`, assert `0.2.1`), plus the new tests qa writes.
- `README.md`: the `## Handoffs through files` section only (lines 111-116), one sentence added to the paragraph that starts "Also in the rule:".
- `/Users/kpatronas/Documents/proj/claude-model-split.md` (outside the repo): the `### Parallel work` section (lines 184-190) gets one paragraph; the `## Keeping the team efficient` list (lines 239-243) gets two bullets. Nothing else in that file.
- `/Users/kpatronas/Documents/proj/claude-model-split.md`, `### When to use it` (line 268): one sentence added, see Constraints.
- Nothing else. `docs/example-session.txt`, `docs/spec.md`, the bench prompts and `README.md:123` (a real transcript banner) stay as they are.

## Constraints
- Routing block: insert this sub-bullet verbatim immediately after the `Parallel work` sub-bullet (after the line ending `are always sequential.`), at the same two-space indent, before the `- Handoffs carry context` bullet:

```
  - Cost: a subagent costs the same alone or beside another; parallel saves time, not tokens.
    What costs is the number of spawns (each starts cold), the model each one runs on, and the
    length of every transcript, because each tool call re-sends the whole transcript so far.
    Agents do not pay for talking to each other; handoffs are one prompt in, one line back.
    So: one scout per ticket with three or four questions, not one broad sweep; `/compact` once
    a ticket closes so the next task starts from the ticket folder, not the conversation; Opus
    (`senior`, `reviewer`) only where judgement is needed. The ladder is for features: a change
    smaller than its spec goes to one `junior` without qa or reviewer, or the architect makes it
    in place, because each handoff is a cold start that re-reads the same files.
```

- README sentence, appended to the "Also in the rule:" paragraph after "so the architect does not.": ` And a cost note: parallel saves time, not tokens, because each agent pays for its own transcript on every tool call, so keep scout questions to three or four per ticket, `/compact` once a ticket closes, and keep the ladder for features: a change smaller than its spec goes to one junior or is made in place.`
- Article, `### Parallel work`: insert this paragraph after the paragraph that ends "merge afterwards." and before "The architect does not write code.":

```
Parallel saves time, not tokens. A subagent costs the same alone or beside another, because the cost is per agent, not per minute, and the agents do not pay for talking to each other: a handoff is one prompt in and one line back, and the reports go through files. What each agent pays for is its own reading. Every tool call is a new request that carries the whole transcript so far, so a scout that answers eleven questions over 147 tool calls costs about 300k tokens whether or not a junior is working next to it. The main session is a transcript too, and the longest one of the day paid for its full length on every turn.

The same arithmetic says when not to use the ladder. Each handoff is a cold start that re-reads the spec and the files, so a ticket that passes through the scout, QA, the builder and the reviewer pays for four readings of what one session would read once. On a feature that is cheap insurance. On a one-file fix it is most of the bill, and slower too. If the spec would take longer to write than the change, give it to one junior without QA or review, or make the change in place.
```

- Article, `## Keeping the team efficient`: add these two bullets after the **Facts outlive tickets** bullet and before **The log makes patterns visible**:

```
- **Narrow scout questions.** One scout per ticket with three or four questions, not one sweep that answers eleven. A broad sweep reads most of the codebase and then re-sends all of it on every later step. The next ticket's scout starts from `FACTS.md` anyway.
- **Compact between tickets.** The main session re-reads its whole conversation every turn. Once a ticket is closed, `/compact`. The next task starts from the ticket folder, which is the point of keeping handoffs in files.
```

- Article, `### When to use it`: in the single paragraph, after the sentence ending "most of your requests are concrete changes.", insert: ` Even then, size the process to the task: the ladder is for features, and a chore smaller than its spec is cheaper done by one junior or in place.`
- Keep the surrounding text byte-identical. Match the existing voice: plain, short sentences, no emoji, no em-dashes.
- Do not run `claude-stuffer init` against the real home directory. The architect does that after review.
- The article folder is not a git repo. Copy the file to the scratchpad first and paste `diff -u` of before and after into build.md.

## Done means
- `./claude-stuffer --version` prints `0.2.1`.
- `HOME=<tmpdir> ./claude-stuffer init` writes a `~/.claude/CLAUDE.md` whose rules block contains the exact Cost sub-bullet above, placed after the Parallel work bullet and before the Handoffs bullet.
- `git diff --stat` in the repo shows only `claude-stuffer`, `tests/test_stuffer.py`, `README.md` (and the ticket folder).
- The article diff shows only the two paragraphs, two bullets and one sentence added.

## Tests that must pass
- `python3 -m unittest discover -s tests -v`: every existing test (88) plus the new ones from `tests.md`. The new tests should check: the rules block contains "- Cost:" and "`/compact`"; the Cost bullet appears after the text "are always sequential." and before "- Handoffs carry context"; `--version` prints `0.2.1`.

## Notes
- Source of the numbers: the user's Claude Code usage panel on 2026-10-10 (scout: 147 tool calls, about 300k tokens; 77% of usage at over 150k context). They are reported as observed, in the article only.
- Report to `docs/tickets/cost-rule/build.md`; reply in chat with one line: verdict plus path.
