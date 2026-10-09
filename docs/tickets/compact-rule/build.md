# build: compact-rule

## senior (Opus)

Verdict: PASS. 115/115 tests pass (`python3 -m unittest discover -s tests -v`). Nothing committed.

### What changed
- `claude-stuffer`
  - VERSION 0.2.6.
  - Cost bullet: the `/compact` clause now reads "`/compact` once a ticket closes or once the context
    passes 150k, whichever comes first, so the next task starts from the ticket folder, not the
    conversation (the optional `~/.claude/hooks/context-guard.py` hook reminds you); never poll a
    running builder: wait for its completion notice, background agents report back on their own and
    every check re-sends the whole transcript; Opus ...". The rest of the bullet and its position are unchanged.
  - New `HOOK_NAME`, `HOOK_BODY` (the hook script, embedded as a raw string), `HOOK_SNIPPET`,
    `hook_text()` (shebang + `# <marker>` + body), `hook_snippet(hook)` (path JSON-quoted and shell-quoted).
  - `cmd_init`: plans `<home>/.claude/hooks/context-guard.py` with `plan_file` (same write/keep/--force/
    not-ours rules as the bench, same `CLAUDE_STUFFER_HOME` override), chmods it 0755 when ours, and
    prints the `UserPromptSubmit` snippet after a real run and after `--dry-run`. Never reads or writes settings.json.
  - `is_ours`: for files starting with `#!`, looks for the marker as a `# ` comment on line 2. Needed
    so `--force` can rewrite the hook and a user's own script at that path is left alone. Agent .md
    files are unaffected (they never start with `#!`).
- `tests/test_stuffer.py`: qa's `CompactRule` kept as written, except the weak
  `test_hook_uses_last_assistant_message_only`, now tightened (300k earlier + 50k last is silent; 50k
  earlier + 300k last prints "300k"). Added: `test_hook_skips_zero_usage_and_sidechain_messages`,
  `test_hook_reads_large_transcript_from_the_end` (~3.3 MB file across 1 MB read chunks, exact line),
  `test_init_force_rewrites_hook_but_leaves_foreign_hook_alone`.
- `README.md`: "What writes where" table, the init paragraph, the cost note (two triggers, no polling),
  a new "## Context guard" section (measurement, hook behaviour, threshold env, snippet), and Undo.
- `~/Documents/proj/claude-model-split.md`: cost paragraph (line ~190, not ~115 as the spec guessed)
  gets the 59%/86%/24% measurement; the "Compact between tickets" bullet (~249) now names both
  triggers, the no-polling rule and the optional hook.

### Hook contract decisions (judgement calls)
- Reads the transcript backwards in 1 MB chunks, so a multi-MB session costs only the tail.
- "Last assistant message" skips entries with all-zero usage (synthetic API-error/interrupt messages)
  and `isSidechain: true` entries, so neither hides the real context size.
- Token count shown as round(n/1000)k; the threshold in the message is the effective one
  (e.g. "> 10k" when overridden). Invalid or non-positive `CLAUDE_CONTEXT_GUARD` falls back to 150000.
- Any exception -> no output, exit 0. Plain-text stdout, not JSON, as the spec asks.

### Hooks docs source
Fetched https://code.claude.com/docs/en/hooks.md on 2026-10-10. Verified:
- Config shape: `{"hooks": {"<Event>": [{"matcher"?, "hooks": [{"type": "command", "command": ...}]}]}}`;
  `UserPromptSubmit` has no matcher support (matcher ignored), so the snippet omits it.
- Input: common fields include `transcript_path` (plus `session_id`, `cwd`, `permission_mode`,
  `hook_event_name`, `prompt`). Docs note the transcript is written asynchronously and may lag the
  current turn; harmless here (the previous turn's usage is what we want).
- "Exit code 0": for `UserPromptSubmit` (also UserPromptExpansion, SessionStart, PostModelSwitch)
  "Claude Code adds plain-text stdout as context that Claude can see and act on"; injected as a
  system reminder, no visible transcript entry. Exit 2 / `decision: "block"` would block the prompt,
  which the hook never does.
- Default timeout for UserPromptSubmit command hooks is 30 s; on timeout output is discarded and the
  prompt proceeds. The hook reads only the file tail, well under that.
- UserPromptSubmit also fires when a background subagent reports back, so the reminder reaches the
  architect at exactly the moment a ticket tends to close.

### Verification (temp home, CLAUDE_STUFFER_HOME=HOME=<scratch>/home)
- `--version` -> `claude-stuffer 0.2.6`.
- `init --dry-run` lists `would write home/.claude/hooks/context-guard.py`, prints the snippet,
  home stays empty.
- `init` writes the hook (-rwxr-xr-x), prints the snippet, no settings.json created; CLAUDE.md
  contains "never poll a running builder".
- Hook: 200k transcript -> `Context is 200k tokens (> 150k). Finish or close the current ticket, then /compact.`
  exit 0; 50k -> nothing, exit 0; missing transcript -> nothing, exit 0.
- Against three real session transcripts in ~/.claude/projects (threshold 1000): 50k, 67k, 207k, exit 0.

### Left open
- Not done (out of scope per spec): applying the block to ~/.claude/CLAUDE.md, adding the hook to the
  user's settings.json, Ghost republish, commit/push.
- The snippet prints the absolute path of the written hook; README shows `/Users/you/...` as a placeholder.
