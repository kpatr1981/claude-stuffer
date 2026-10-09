
## mid build

Verdict: done. Docs only.

### Changed
- README.md `## Install`: Python 3.9+ sentence and xcode-select hint; primary block (`curl -fsSL ... -o /tmp/claude-stuffer`, `sudo install -m 0755 /tmp/claude-stuffer /usr/local/bin/claude-stuffer`, `claude-stuffer init`); one clause saying macOS and Linux are the same; "Without sudo" Homebrew block (`"$(brew --prefix)/bin"`) and `~/.local/bin` block with the `~/.zshrc` PATH line. The paragraph after (`init` writes...) is untouched (diff shows no change to it).
- docs/spec.md lines 254-255: curl/chmod replaced by curl to /tmp + sudo install.
- /Users/kpatronas/Documents/proj/claude-model-split.md: same two lines replaced; block matches README primary block exactly.

### Verification
`ls -ld /usr/local/bin`: `drwxr-xr-x  29 root  wheel  928 Oct 10 00:58 /usr/local/bin`

`curl -fsSL <raw url> -o /tmp/cs-test && python3 /tmp/cs-test --help`: curl rc=0, usage printed (`usage: claude-stuffer [-h] [--version] [--kinds] [--dry-run] ... {init,show,ticket} ...`), rc 0. /tmp/cs-test removed afterwards.

`sudo install` was NOT run (needs the user's password).

Tests: `python3 -m pytest tests -q` could not run, pytest is not installed on this Mac ("No module named pytest"). Ran the project's own runner instead: `python3 -m unittest discover -s tests` -> Ran 88 tests, OK.

`git diff --stat`:
```
 README.md             | 26 ++++++++++++++++++++++++--
 docs/spec.md          |  4 ++--
 docs/tickets/FACTS.md |  3 +++
```
FACTS.md was already modified by the scout before this build; I did not touch it. My edits are README.md and docs/spec.md only.

Article diff (`diff -u` against the scratchpad .orig copy):
```
@@ -64,8 +64,8 @@
 ```bash
-curl -fsSL https://raw.githubusercontent.com/kpatr1981/claude-stuffer/main/claude-stuffer -o /usr/local/bin/claude-stuffer
-chmod +x /usr/local/bin/claude-stuffer
+curl -fsSL https://raw.githubusercontent.com/kpatr1981/claude-stuffer/main/claude-stuffer -o /tmp/claude-stuffer
+sudo install -m 0755 /tmp/claude-stuffer /usr/local/bin/claude-stuffer
 claude-stuffer init
```

### Open
- Not run under zsh and bash separately; commands are plain curl/install/mkdir/echo with no shell-specific syntax.
- Not verified on a Python 3.9 interpreter (scout noted the same).
