PASS

# review: compact-rule (reviewer, Opus, 2026-10-10)

## 1. Spec coverage
- Rule text, claude-stuffer:503-507: the `/compact` clause names both triggers ("once a ticket closes or once the context passes 150k, whichever comes first") and adds "never poll a running builder: wait for its completion notice". The rest of the Cost bullet and its position are unchanged. The old tests at tests/test_stuffer.py:586-600 pass unmodified.
- Hook, claude-stuffer:192-279 (HOOK_BODY): stdlib only. It reads the stdin JSON and `transcript_path`, and sums input + cache_read + cache_creation from the last `type: assistant` line. It prints one line over the threshold, which is 150000 or `CLAUDE_CONTEXT_GUARD`. `main()` wraps everything in `except Exception` and always returns 0.
- init, claude-stuffer:1234-1248: the hook goes through `plan_file`, so it gets the same keep/--force/not-ours/symlink rules and the same `CLAUDE_STUFFER_HOME` override as the bench. The file is chmod 0755 only when it is ours. init prints the snippet after both a dry run and a real run, and the snippet says init does not edit settings.json. Nothing in the diff reads or writes settings.json.
- VERSION is 0.2.6 (claude-stuffer:25), and `--version` prints `claude-stuffer 0.2.6`.
- README.md:40, 77, 115-133 and 212 cover the two triggers, the no-polling rule, the "Context guard" section with the snippet, and Undo. In the article ~/Documents/proj/claude-model-split.md, line 190 has the measurement and line 249 has the triggers, the no-polling rule and the optional hook. The cost paragraph is at line 190, not ~115 as the spec said; build.md notes this.
- Extra change, claude-stuffer:588-590: `is_ours` now recognises a marker comment under a shebang. The spec did not list this, but `--force` and the not-ours protection on the hook need it, and it only affects files that start with `#!`. Agent .md files are unaffected. Accepted as part of "init writes it".
- Scope: the diff touches only README.md, claude-stuffer and tests/test_stuffer.py in the repo, plus the article outside it. No bench agent files and no other rules changed.

## 2. Tests
- I ran `python3 -m unittest discover -s tests`: 115 tests, OK. That is the full suite, not a subset: the 96 existing tests (the version test renamed to 0_2_6) plus 19 new ones in `CompactRule`. The only removed lines in the test diff are the old 0.2.5 version test.
- I also checked by hand in a scratch home:
  - `init --dry-run` lists the hook and prints the snippet, and the home stays empty.
  - A real `init` writes the hook as -rwxr-xr-x. An existing settings.json is byte-identical afterwards (cmp).
  - Hook results:
    - 200k prints the reminder.
    - 50k prints nothing.
    - These inputs all print nothing and exit 0: a missing path, empty stdin, `[]` stdin, a directory as the path, a non-UTF8 garbage transcript, and /dev/null stdin.
    - `CLAUDE_CONTEXT_GUARD=abc` falls back to 150k.

## 3. Safety
- The hook cannot block a prompt: every path exits 0, and it never uses exit 2 or JSON `decision`. It only reads the transcript tail, in 1 MB chunks, so it stays far under the 30 s hook timeout.
- No secrets, no network access, and settings.json is never touched.
- The hook is never written through a symlink, and it is never written over a script that is not ours (plan_file:896-901).

## 4. Would approve. Non-blocking nits, for a later ticket if wanted
- claude-stuffer:882 (`hook_snippet`): the path goes into shell double quotes, so a home path containing `$`, a backtick or `"` would be expanded or break the command. Realistic home paths are fine. Fix: use `shlex.quote(hook)` instead of `"%s"`.
- HOOK_BODY `kilo()`: tokens from 150001 to 150499 print "Context is 150k tokens (> 150k)". This is cosmetic. Fix: print one decimal near the threshold, or compare against the rounded value.
- cmd_init, claude-stuffer:1241/1248: when the hook path holds a foreign script ("not ours"), init still prints the snippet pointing at that script. Fix: print the snippet only when the action is not "not ours".
