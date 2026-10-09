# macos-install

## What
The README's Install section gives one `curl ... -o /usr/local/bin/claude-stuffer` command that fails on a stock Mac: `/usr/local/bin` is owned by `root:wheel` (mode 755), so the download is refused with "Permission denied". Replace the section with install instructions that actually work on macOS, keep a Linux path that works too, and state the Python requirement. The article draft carries the same block and gets the same fix.

## Files
- `README.md` (the `## Install` section only, lines 47-55 as of commit 7291262)
- `docs/spec.md` (only the two `curl`/`chmod` install lines at 254-255; replace them with the new primary block)
- `/Users/kpatronas/Documents/proj/claude-model-split.md` (the one fenced `curl` block under "## Setting it up: claude-stuffer", lines 65-69; nothing else in that file)

## Constraints
- Docs only. Do not touch `claude-stuffer`, `tests/`, or any other README section.
- Verified facts about macOS, use them as-is:
  - `/usr/local/bin` is `root:wheel` 755 on macOS, so writing there needs `sudo`. Use `sudo install -m 0755 <downloaded file> /usr/local/bin/claude-stuffer` (`install` creates the file with the right mode in one step; it does not create the directory, and `/usr/local/bin` exists on macOS).
  - On Apple Silicon with Homebrew, `/opt/homebrew/bin` is user-writable and on PATH; on Intel Homebrew it is `/usr/local/bin`. Offer `"$(brew --prefix)/bin"` as the no-sudo option for people who have Homebrew; it resolves to the right prefix on both.
  - `~/.local/bin` is a valid no-sudo option but is NOT on PATH by default in macOS zsh; if you show it, show the `export PATH="$HOME/.local/bin:$PATH"` line for `~/.zshrc` next to it.
  - macOS ships `/usr/bin/python3` as a shim that needs the Xcode Command Line Tools; without them the first run prints an install prompt. One sentence: if `python3 --version` prompts to install tools, run `xcode-select --install` once.
  - `curl` does not set the quarantine attribute, so no Gatekeeper step is needed. Do not add `xattr` commands.
- Layout: one primary recipe (download to a temp file with `curl -fsSL -o`, then `sudo install`), then a short "Without sudo" alternative, then `claude-stuffer init`. Linux users get the same primary recipe; say so in one clause rather than a second block. Keep the existing paragraph that follows the code block (about `init`, `--force`, and the Fable model setting) unchanged.
- Python floor: say "Python 3.9 or newer". That is the declared contract in `docs/spec.md:9`; the scout (`docs/tickets/macos-install/scout.md`) found no code needing anything newer than 3.4, so 3.9 is safe to promise. Do not change the README intro line.
- Style: match the README's voice (plain, short sentences, no emoji, no "simply"). Fenced blocks tagged `bash`. The command lines must be copy-pasteable as written.
- The article block gets the same primary recipe (the `curl` + `sudo install` + `init` lines) and nothing else; the article does not get the alternatives paragraph.

## Done means
- A fresh macOS user who pastes the primary block gets `claude-stuffer` on PATH and `claude-stuffer init` runs. The builder verifies this on this Mac by running the primary block with the download redirected to a scratch file and `sudo` replaced by a dry step: `curl -fsSL <raw url> -o /tmp/cs-test && python3 /tmp/cs-test --help` must print usage. (`sudo install` itself is not run by the builder; it needs the user's password. Say so in build.md.)
- `/usr/local/bin` ownership claim is backed by `ls -ld /usr/local/bin` output pasted in build.md.
- `shellcheck`-level sanity: every command in the blocks runs under both zsh and bash without edits.
- The paragraph after the Install block is byte-identical to before.
- The article file's block matches the README's primary block exactly.

## Tests that must pass
- `python3 -m pytest tests -q` stays green (no code touched; this proves nothing was accidentally edited).
- `git diff --stat` shows only `README.md` and `docs/spec.md` in this repo. The article lives outside the repo; that folder is not a git repo, so copy it to `/private/tmp/claude-501/-Users-kpatronas/fa702b0e-512e-4241-978b-15c8f1b772a9/scratchpad/claude-model-split.md.orig` first and paste `diff -u` of the two into build.md.

## Notes
- Verified on this Mac (Darwin 25.6.0, 2026-10-10): `/usr/local/bin` → `drwxr-xr-x root wheel`; `/opt/homebrew/bin` → `drwxrwxr-x kpatronas admin`; `~/.local/bin` exists and is on PATH here only because the user added it.
- The raw URL: `https://raw.githubusercontent.com/kpatr1981/claude-stuffer/main/claude-stuffer`.
- Report to `docs/tickets/macos-install/build.md`; reply in chat with one line: verdict plus path.
