# claude-stuffer: spec

A single-file command that staffs a Claude Code project with a team of subagents, following the
"team by seniority" method (architect Fable, senior Opus, mid Sonnet, junior Haiku; roles beside
the ladder; per-project domain roles). Repo: github.com/kpatr1981/claude-stuffer.

## Constraints

- One executable file `claude-stuffer`, Python 3.9+, standard library only, shebang `#!/usr/bin/env python3`.
- Works on macOS and Linux. No network access.
- Never deletes a file. Never overwrites a file it did not generate unless `--force`.
- Tests: `tests/test_stuffer.py`, stdlib `unittest`, run with `python3 -m unittest discover -s tests -v`
  from the repo root. Tests import the script by path (`importlib` on the file `claude-stuffer`) and
  also run it as a subprocess for the CLI cases. Tests use temp dirs only; nothing touches the real `~`.
- Exit codes: 0 ok, 1 user abort or nothing to do, 2 bad usage or error.

## CLI

```
claude-stuffer                      interview in the current folder, then write the team
claude-stuffer init                 install or update the global bench in ~/.claude
claude-stuffer show                 print the project's current team
claude-stuffer --kinds              list project kinds and the roles each proposes
claude-stuffer --dry-run            interview, print every file that would be written, write nothing
claude-stuffer --answers FILE.json  non-interactive: take answers from FILE (see "Answers file")
claude-stuffer --project DIR        use DIR instead of the current folder
claude-stuffer --force              overwrite agent files this tool generated earlier
claude-stuffer --version / --help
```

`init` honours `--dry-run` and `--force`. The environment variable `CLAUDE_STUFFER_HOME`, when set,
replaces `~` for `init` (tests use it). `init` and the interview can be combined: `claude-stuffer init`
first, then `claude-stuffer` in a project; the interview warns (does not fail) if the bench is missing.

## Levels and models

| level  | model  |
|--------|--------|
| senior | opus   |
| mid    | sonnet |
| junior | haiku  |

The architect is the main session and is never generated.

## Detection (runs before the interview, provides defaults)

From the project folder, ignoring `.git`, `node_modules`, `venv`, `.venv`, `__pycache__`, `build`, `dist`:

- **Languages/frameworks**: `pyproject.toml`/`requirements*.txt`/`*.py` → Python; `go.mod` → Go;
  `pubspec.yaml` → Flutter/Dart; `package.json` → JavaScript/Node; `Cargo.toml` → Rust;
  `*.sh`/`*.bash` → Shell; `*.ksh` → ksh; `project.godot` → Godot; `docker-compose*.yml`/`Dockerfile` → Docker;
  `*.sql` → SQL; `*.ps1` → PowerShell. Deduplicated, in that order.
- **Test command**: first match wins: `go.mod` → `go test ./...`; `pubspec.yaml` → `flutter test`;
  `package.json` with a `scripts.test` → `npm test`; `tests/*.bats` or `test/*.bats` → `bats tests` (or `test`);
  `Makefile` with a `test:` target → `make test`; `pytest.ini`/`pyproject.toml`/`tests/*.py`/`test_*.py` → `pytest`;
  else none.
- **Kind guess**: `project.godot` or a folder named `godot`/`genres`/`games` → game; `pubspec.yaml` or
  `android/` + `ios/` → mobile; `docker-compose*.yml` or `Caddyfile`/`nginx*.conf` or folder `ops`/`deploy` → infra;
  `package.json` with `react`/`vue`/`svelte` in dependencies, or `index.html` + `*.css` → web;
  `*.sql` ≥ 3 or folders `data`/`etl` → data; `go.mod`/`pyproject.toml` with a `main.go`/`cli.py`/`__main__.py` → cli;
  `setup.py`/`pyproject.toml` with no `main`, or `lib/` + `go.mod` → library; mostly `*.md` (not counting `README.md` and `CLAUDE.md`) → content;
  else other. First rule that matches wins, in this order.
- **Docs to read first**: existing among `README.md`, `CLAUDE.md`, `docs/*.md`, `docs/**/*.md`,
  anything matching `*style*guide*.md`, `*conventions*.md`, `*architecture*.md`; at most 8, README and CLAUDE.md first.
- **Production hint**: true if any file under `ops/`, `deploy/`, a `Caddyfile`, `docker-compose*.yml`,
  `*.ksh`, or a file containing the word `cron` in `*.sh`/`*.ksh`/`Makefile` exists.
- **Description default**: first non-heading paragraph of `README.md`, first sentence, max 200 chars; else empty.

## Interview (stdin/stdout; Enter accepts the default shown in brackets)

1. Project name `[folder name]`
2. One line: what is it? `[README sentence or empty]` (required; re-ask if empty)
3. Kind: numbered list of kinds, `[detected]`
4. Languages/frameworks, comma separated `[detected]`
5. Test command `[detected or "none"]`. If none, print: "No tests: code changes will be routed to the senior and every spec must say how to verify."
6. Touches production systems (servers, cron, live databases)? `[y/N, default from hint]`
7. Documents a role must read first, comma separated `[detected]`
8. Proposed roles for this kind, one by one: `Keep <name> (<level>): <one line>? [Y/n]`
9. Add-ons, each `[y/N]`: `site-ops` if production and not already proposed; `security-reviewer` if production,
   or if the description or languages mention auth, token, secret, login, payment, ssh, aws, api.
10. Custom roles, repeated until blank name: name (lowercase, hyphens), what it does (one line),
    what it must know (comma-separated paths or notes), level `[mid]`.
11. Summary of files to write, then `Write these files? [Y/n]`. `n` exits 1 without writing. **End of input at this
    question means no**: exit 1 with "no confirmation (end of input), nothing written". Earlier questions may take
    their defaults at end of input; the confirmation never does.

Interactive answers are validated: role names `^[a-z][a-z0-9-]{1,40}$`; levels one of senior/mid/junior.

## Answers file (non-interactive)

```json
{
  "name": "logsieve",
  "description": "tails and filters logs from several hosts over SSH",
  "kind": "cli",
  "languages": ["Go"],
  "test_command": "make test",
  "production": true,
  "docs": ["README.md", "docs/design.md"],
  "roles": ["cli-developer"],
  "addons": ["security-reviewer"],
  "custom_roles": [
    {"name": "ksh-engineer", "level": "senior", "description": "legacy ksh88 scripts on prod boxes",
     "knows": ["scripts/README.md", "ksh88 only, no bashisms"]}
  ]
}
```

Missing keys take the detected defaults; `roles` missing means all proposed roles; `addons` missing means none;
`--answers` implies no prompts at all and no confirmation question. Unknown kind, role or add-on name → exit 2 with a message.
Types are checked: list fields accept a list of strings or a single string; `production` accepts only JSON true/false;
anything else (a number, a boolean where a list is expected, the string "false" for production) → exit 2 with a message, before any write.

## Catalog: kinds → proposed roles

Each role has: name, default level, one-line description, and 2–4 rule sentences that go into its prompt.

- **cli**: `cli-developer` (senior): every flag in `--help`; stable exit codes; a non-interactive mode for everything the interactive mode does; never a second code path for the same operation.
- **service**: `backend-developer` (senior): every endpoint has a test; input validated at the boundary; errors are typed, not strings. `db-engineer` (mid): schema changes only through reversible migrations; every query comes with the question it answers and its plan on realistic data.
- **web**: `frontend-developer` (mid): components are small and tested; no inline styles outside the design system; accessibility (contrast, keyboard, labels) is part of done. `backend-developer` (senior) as above. `ui-designer` (mid): screens as text and code (components, SVG); states (empty, loading, error) always specified; cannot paint, writes briefs for raster assets.
- **mobile**: `app-developer` (senior): business logic out of widgets so it can be unit tested; permissions requested at point of use with a reason; platform folders change only when the spec names a plugin. `ui-designer` (mid) as above.
- **game**: `game-designer` (senior): every change stated in numbers (frames, pixels, hit points, rates) with how to measure it; playability claims need a solver or scripted run; balance against a skilled-player baseline. `gameplay-programmer` (senior): rules identical across every target unless the spec says otherwise; respect the platform's budget (memory, frame time); tests for the rules, scripted runs for feel. `artist` (mid): art as data the build can consume; palette and size constraints respected; cannot paint, writes briefs and judges results.
- **data**: `data-engineer` (senior): loads are idempotent and re-runnable; schema is versioned; every transformation has a row-count or checksum check. `analyst` (mid): every query states the question it answers; numbers are checked against a known total before being reported; no chart without the table behind it.
- **infra**: `site-ops` (mid; senior for production changes): read and diagnose freely, change only what the spec says; back up first (timestamped copy, DB dump before schema or version changes); one change, verify, state the rollback; never put secrets in a report.
- **library**: `library-developer` (senior): public API stays stable within a major version; every public symbol documented; examples in docs are tested. `docs-writer` (mid): documents from the code, not from memory; every example runnable; one concept per page.
- **content**: `tech-writer` (mid): plain, direct, problem first, no padding; every command shown is run before it is written; follows the project's style guide if one exists. `editor` (senior): edits for accuracy and voice, cuts what does not earn its place, lists every change and why.
- **other**: no proposed roles; custom only.

Add-ons: `security-reviewer` (senior, read-only, tools `Read, Grep, Glob, Bash`): secrets or tokens on a command line, in logs, or in saved configs; trust boundaries and input validation; dependency fetching (pinning, checksums, HTTPS); file permissions on credentials. Reports PASS or FAIL first, findings most severe first, each with file, line, severity and a concrete fix; never edits. `site-ops` as in infra.

## Generated project files

For each role, `<project>/.claude/agents/<name>.md`:

```
---
name: <name>
description: <Role title> for <project> (<level>, <model>; level set per task by the architect). <one line>.
model: <model>
tools: <only for read-only roles>
---
<!-- generated by claude-stuffer on YYYY-MM-DD. Edit freely; re-running overwrites this file only with --force. -->
You are the <role title> for <project>: <project description>.
Read first: <docs, comma separated>.            (omitted if none)
Stack: <languages>.                             (omitted if none)
Tests: run `<test command>` after every change.  OR
No test suite here: verify the way the spec says (run it, hit it, check the output) and include that output in your report.
Production: this project touches live systems. Change nothing the spec does not name; back up first; one change at a time; verify; state the rollback.   (only if production)
Rules:
- <catalog rule 1>
- ...
If tests or checks fail: analyze and fix, two attempts. If still failing, stop, leave your changes in place and report what failed (actual output), what you tried, your hypothesis, whether it looks mechanical or a logic problem, and which files are modified.
Report on success: what changed, what you ran and its output, anything left open.
```

Role title = name with hyphens replaced by spaces. Custom roles use their `knows` list as "Read first/Know:" lines and have no catalog rules. Read-only roles (security-reviewer) replace the failure paragraph with "You never modify a file. If you see a fix, describe it." and the report line with their own.

`<project>/CLAUDE.md`: created with `# <name>` if missing. A block delimited by
`<!-- claude-stuffer:team -->` and `<!-- /claude-stuffer:team -->` is inserted at the end or replaced in place:

```
## Team (staffed YYYY-MM-DD by claude-stuffer)

Global bench applies (`senior`, `mid`, `junior`, `reviewer`, `qa`, `scout`). Project roles, level chosen per task by the architect:

- `<name>` (<Model>; <level>): <one line>.
- ...
- No test suite: code changes go to the most senior role that fits, and every spec says how to verify.   (if none)
- Production: changes are specified, backed up and verified one at a time; the user applies anything the spec marks as manual.   (if production)
```

Re-run behaviour: agent files that exist are skipped (listed as "kept") unless `--force`; files without the generated-by marker are never overwritten even with `--force` (listed as "not ours"). "Ours" means the file's first body line starts with the full marker `<!-- generated by claude-stuffer on `; a file that merely mentions the tool is not ours. A symlink is never ours. A CLAUDE.md with a start marker and no end marker (or the reverse) is an error: exit 2 "unbalanced claude-stuffer markers", nothing written. The CLAUDE.md block is always replaced. Roles not chosen this time are not deleted; the summary prints "remove by hand: <paths>" if such generated files exist.

## `init`: the global bench

Writes to `$HOME/.claude/agents/`: `senior.md`, `mid.md`, `junior.md`, `reviewer.md`, `qa.md`, `scout.md`.
Their contents are embedded in the script verbatim from the current files in `~/.claude/agents/` of the
author's machine (copy them from `/Users/kpatronas/.claude/agents/*.md` at build time; they are the source of truth).
Existing files are kept unless `--force` (same marker rule; the bench files carry the same generated-by comment).

Inserts or replaces in `$HOME/.claude/CLAUDE.md` a block between `<!-- claude-stuffer:rules -->` and
`<!-- /claude-stuffer:rules -->` containing the routing rule. The rule text is the content of
`/Users/kpatronas/.claude/CLAUDE.md` below its `# Global instructions` heading, with the staffing bullet's
first sub-item changed to: "When a project starts, or has no `.claude/agents/` of its own, run `claude-stuffer`
in its folder; it asks about the project and writes the team. Confirm the proposed roles with the user before writing."
Creates `CLAUDE.md` with `# Global instructions` if missing.

After writing, prints: the files written/kept, then
"Default model: set `\"model\": \"claude-fable-5-1\"` in ~/.claude/settings.json or run /model claude-fable-5-1 in a session.
Check with /agents in a new session."

## `show`

Reads `<project>/.claude/agents/*.md`, prints a table: name, model, level (derived from model), the role's one-line
description (for generated files, the text after the `architect). ` prefix; otherwise the first sentence), and whether the file was generated by this tool. Exit 1 with "no team here, run claude-stuffer" if the folder is missing or empty.

## Repo contents

```
claude-stuffer          the script (executable)
tests/test_stuffer.py
docs/spec.md            this file
README.md               what it is, install, usage, what it writes, how to undo, link to the article
LICENSE                 MIT, Copyright (c) 2026 Konstantinos Patronas <kpatronas@gmail.com>
```

README install section:

```
curl -fsSL https://raw.githubusercontent.com/kpatr1981/claude-stuffer/main/claude-stuffer -o /usr/local/bin/claude-stuffer
chmod +x /usr/local/bin/claude-stuffer
claude-stuffer init
cd ~/code/my-project && claude-stuffer
```

Undo: delete `<project>/.claude/agents/` files it generated and the marked block in CLAUDE.md; same for `~/.claude` after `init`.

## Done means

- `python3 -m unittest discover -s tests -v` passes.
- `./claude-stuffer --kinds`, `--version`, `--help` run with exit 0.
- `CLAUDE_STUFFER_HOME=$(mktemp -d) ./claude-stuffer init` writes 6 agents and the rules block; a second run keeps all 6 and leaves exactly one block.
- `./claude-stuffer --project <tmp copy of a Go project> --answers a.json` writes the agents and the Team block; `--dry-run` writes nothing; `show` lists them.
- `python3 -m py_compile claude-stuffer` is clean; no third-party imports.
