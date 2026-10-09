# Project facts (appended by the scout)

- The script is one file, `claude-stuffer`, stdlib only; tests in tests/test_stuffer.py run with `python3 -m unittest discover -s tests -v` (88 tests at v0.2.0).
- The embedded bench and rules text come from ~/.claude/agents/*.md and the marked block in ~/.claude/CLAUDE.md on the author's machine; regenerate them when those change.
- docs/example-session.txt is the real transcript used in README.md and the blog article; regenerate it when prompts change.
