# Scout: Python version, version string, install-location dependence

Read first: docs/tickets/FACTS.md (3 lines of facts, nothing on Python version or install path).
Method: read all of `claude-stuffer` (1237 lines) and `tests/test_stuffer.py` (first 75 lines read in full, rest searched by pattern); grepped the whole repo for f-strings, walrus, `match`, annotations, `removeprefix`, `zoneinfo`, `tomllib`, `datetime.UTC`, `importlib.resources`, `argv`, `__file__`, `/usr/local`, `version`, `Python`. Not run on any interpreter, so no version is confirmed by execution.

## 1. Minimum Python 3 version

Shebang: `/Users/kpatronas/Documents/proj/personal/tools/claude-stuffer/claude-stuffer:1` is `#!/usr/bin/env python3` (no minor version pinned).

Searched constructs, none found in the script:
- walrus `:=`: none (the only `:=`-like text is inside strings, none is code).
- `match` statement: none.
- f-strings: none. Formatting is `%` throughout (e.g. `claude-stuffer:937`, `claude-stuffer:1222`).
- annotations (`->`, `: str`, `str | None`): none, so no runtime-evaluated unions and no `from __future__` needed.
- `removeprefix`/`removesuffix`, `dict | dict`, `list[str]` generics, `tomllib`, `zoneinfo`, `datetime.UTC`, `importlib.resources`: none.
- argparse: `action="version"` (`claude-stuffer:1194`), `add_subparsers(dest=..., metavar=...)` (`claude-stuffer:1197`), `parse_args`. Nothing newer than long-standing 3.x argparse features; no `BooleanOptionalAction` or `parse_intermixed_args`.
- Other calls: `os.makedirs(..., exist_ok=True)` (`claude-stuffer:436`), `re.fullmatch` (`claude-stuffer:817`), `os.path.lexists` (`claude-stuffer:1105`).

Highest requirement found: the script itself needs nothing newer than roughly Python 3.4-era stdlib by the constructs above. The only construct in the repo that needs 3.7+ is in the tests, not the script: `subprocess.run(..., capture_output=True, text=True, timeout=60)` at `tests/test_stuffer.py:65-66`.

The declared floor is Python 3.9+: `docs/spec.md:9` "Python 3.9+, standard library only". I found no code line that needs 3.9; that number is a stated requirement, not something the code demonstrates. Whether the script runs on 3.9 or older was not tested.

Local interpreter evidence only: `tests/__pycache__/test_stuffer.cpython-314.pyc` exists, so the author ran tests on CPython 3.14. Not a requirement.

## 2. Version flag and version string

Yes. Constant: `claude-stuffer:25` `VERSION = "0.2.0"`.
- `--version` flag: `claude-stuffer:1194` `parser.add_argument("--version", action="version", version="claude-stuffer " + VERSION)`. Prints `claude-stuffer 0.2.0`.
- Interview banner: `claude-stuffer:937` `print("claude-stuffer %s: staffing a project. ..." % VERSION)`.
- Tests: `tests/test_stuffer.py:255` `def test_version_prints_0_2_0(self):` and `:265` `def test_version_exits_zero(self):`, both calling `run(["--version"])`.
- Spec: `docs/spec.md:29` `claude-stuffer --version / --help`; `docs/spec.md:265` `--version`, `--help`, `--kinds` run with exit 0.
- Also in `docs/tickets/FACTS.md:3`: "(88 tests at v0.2.0)".

## 3. Dependence on install location

Script: no `sys.argv`, no `__file__`, no `sys.executable`, no `/usr/local` or `/usr/bin` anywhere in `claude-stuffer`. `prog="claude-stuffer"` is hard-coded (`claude-stuffer:1190`), so argv[0] does not affect usage text. Home-dependent paths come from `stuffer_home()` at `claude-stuffer:467-468`: `os.environ.get("CLAUDE_STUFFER_HOME") or os.path.expanduser("~")`, not from the script's location. Bench files go to `$HOME/.claude/agents` (`claude-stuffer:1040`, `:1082-1083`).

Tests: `tests/test_stuffer.py:18` `ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))` and `:19` `SCRIPT = os.path.join(ROOT, "claude-stuffer")`. Tests run the repo copy of the script, never an installed one. `:65` uses `sys.executable` to launch it. `:52-63` sets `HOME` and `CLAUDE_STUFFER_HOME` to a temp dir. No test references `/usr/local/bin` or `PATH`.

The only mention of `/usr/local/bin` is install docs: `README.md:50` `curl -fsSL https://raw.githubusercontent.com/kpatr1981/claude-stuffer/main/claude-stuffer -o /usr/local/bin/claude-stuffer` and `README.md:51` `chmod +x /usr/local/bin/claude-stuffer`. The same lines are in `docs/spec.md:254-255`. Nothing in code depends on that path.

## 4. Python version mentions

- `README.md:5`: "One Python 3 file, no dependencies. Run it once per machine to install the bench, then once per project to staff it."
- `README.md:177`: "python3 -m unittest discover -s tests -v" (no version given).
- `docs/spec.md:9`: "One executable file `claude-stuffer`, Python 3.9+, standard library only, shebang `#!/usr/bin/env python3`."
- `docs/spec.md:12`: "Tests: `tests/test_stuffer.py`, stdlib `unittest`, run with `python3 -m unittest discover -s tests -v`".
- `tests/test_stuffer.py:3`: "Run from the repo root:  python3 -m unittest discover -s tests -v". No version check in the tests (no `version_info` or `sys.version` anywhere).
- `docs/spec.md:269`: "`python3 -m py_compile claude-stuffer` is clean; no third-party imports."

README gives no minor version. The only numeric floor is `docs/spec.md:9` (3.9+).

## Open for a more senior look
- Whether the script actually runs on 3.9 (or on the macOS system `python3` if that is older) is not verified; it needs a run under that interpreter.
- Whether the 3.9 floor in `docs/spec.md:9` is intended or just a guess is a decision for the architect.
