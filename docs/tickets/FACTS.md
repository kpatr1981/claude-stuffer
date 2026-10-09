# Project facts (appended by the scout)

- The script is one file, `claude-stuffer`, stdlib only; tests in tests/test_stuffer.py run with `python3 -m unittest discover -s tests -v` (88 tests at v0.2.0).
- The embedded bench and rules text come from ~/.claude/agents/*.md and the marked block in ~/.claude/CLAUDE.md on the author's machine; regenerate them when those change.
- docs/example-session.txt is the real transcript used in README.md and the blog article; regenerate it when prompts change.
- The version constant is `VERSION = "0.2.0"` at claude-stuffer:25 (used by `--version` at :1194 and the interview banner at :937); tests for it are tests/test_stuffer.py:255-266.
- Min Python: the declared floor is 3.9+ (docs/spec.md:9, the only number anywhere); no 3.9-only construct found in the script (no walrus, match, f-strings, annotations); the only 3.7+ construct is in tests (capture_output at tests/test_stuffer.py:66). Not run on any old interpreter, so unverified.
- The script does not depend on its install path: no sys.argv or __file__ in claude-stuffer; tests use __file__ (tests/test_stuffer.py:18) to find the repo copy. Install path /usr/local/bin appears only in README.md:50-51.
