# agents-wizard-removed

## What
Claude Code removed the `/agents` wizard. Its removal notice now says: "Ask Claude to create or
update subagents for you, or edit the files directly: .claude/agents/ (this project),
~/.claude/agents/ (all projects)." claude-stuffer still tells users to "Check with /agents in a
new session" after `init` and after staffing a project. Replace that hint everywhere with the
tool's own command, `claude-stuffer show`, which prints the current team. In the README and the
article, also say in one sentence that the wizard is gone and that editing the files is now the
official way, which is exactly what this tool does.

## Files
- `claude-stuffer` (lines ~184 in `INIT_HINT`, and ~1077 in the staffing command's final print)
- `tests/test_stuffer.py` (line ~548, `test_init_prints_default_model_hint`)
- `README.md` (line ~164, the last line of the logsieve example output)
- `/Users/kpatronas/Documents/proj/claude-model-split.md` (line ~134, same example output; line ~273, step 2 of "Checking it works")

## Constraints
- Exact replacement text for the two printed strings and the two example outputs:
  `Check with claude-stuffer show, or start a new session; agents are loaded at startup.`
- In the test, replace the `"/agents"` assertion with `"claude-stuffer show"`. Do not add or remove tests.
- README: directly after the fenced block that ends with the example output (around line 165, before
  "Notice the kind:"), do not add anything. Instead, in the "What writes where" section (line ~36) add one
  sentence at the end of the section: "Claude Code's `/agents` wizard has been removed; the official way
  to add or change subagents is now to edit these files, which is what this tool does."
- Article: step 2 of "Checking it works" becomes: "Run `claude-stuffer show` in the project folder and
  check that the seven bench agents and your project roles are listed." Add the same one-sentence wizard
  note from the README at the end of the paragraph that starts "What it writes is small and readable" (line ~81).
- Touch nothing else. Keep the files' style (plain sentences, no bullet-point additions).
- Commit in the claude-stuffer repo only (the article is outside it): message
  `Point users to claude-stuffer show; the /agents wizard is gone`, ending with the line
  `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`. Then `git push origin main`.
  Do not commit the ticket folder; add nothing under docs/tickets to the commit.

## Done means
- `grep -rn "/agents" claude-stuffer tests README.md` shows no "Check with /agents" line and no `/agents` test assertion. Mentions of the path `.claude/agents` stay.
- `grep -n "/agents" ~/Documents/proj/claude-model-split.md` shows only `.claude/agents` paths and the new wizard-removed sentences.
- Commit pushed; `git status` clean except `docs/tickets/`.

## Tests that must pass
- `python3 -m unittest discover -s tests -q` (run from the repo root; all green, same count as before the change)

## Notes
No scout needed; the architect located every occurrence with grep. Verification run of the
suite before the change is the baseline printed in build.md.
