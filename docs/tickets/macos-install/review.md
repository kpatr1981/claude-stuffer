PASS

# Review: macos-install

Reviewer: reviewer (Opus), 2026-10-10. Read spec.md, scout.md and build.md, `git diff` in the repo, and the article diffed against the scratchpad `.orig` copy.

## 1. Spec conformance

- README.md:49 has the Python floor sentence ("Python 3.9 or newer") and the `xcode-select --install` hint, matching the spec's fact. The intro line README.md:5 is unchanged.
- README.md:51-55 has the primary block: `curl -fsSL ... -o /tmp/claude-stuffer`, then `sudo install -m 0755 /tmp/claude-stuffer /usr/local/bin/claude-stuffer`, then `claude-stuffer init`. It is tagged `bash`. `sudo` is needed: I ran `ls -ld /usr/local/bin` on this Mac and got `drwxr-xr-x root wheel`. `install` sets the mode, so the old `chmod` line is gone, which is correct.
- README.md:57 says in one clause that Linux uses the same recipe, and gives the reason for sudo. There is no second Linux block, as the spec asked.
- README.md:59-64 is the Homebrew no-sudo option, using `"$(brew --prefix)/bin/claude-stuffer"`. The path is quoted. On this Mac `brew --prefix` returns `/opt/homebrew`, and `/opt/homebrew/bin` is `kpatronas admin 775`, so it is writable by the user.
- README.md:66-73 is the `~/.local/bin` option. It says the directory is not on the default zsh PATH and shows the `export PATH="$HOME/.local/bin:$PATH"` line for `~/.zshrc` next to it (:72). It uses single quotes, so `$HOME` and `$PATH` are expanded when the shell starts, not when the line is written. That is correct.
- README.md:75 says "Then run `claude-stuffer init`." This matches the required layout: primary block, then without sudo, then init.
- README.md:77: the paragraph after the block is byte-identical to HEAD:README.md:55 (md5 `da94bab8...` for both).
- There is no `xattr` step.
- docs/spec.md:254-255: the two lines are replaced with the primary recipe. The `claude-stuffer init` and `cd ...` lines after them are kept.
- Article /Users/kpatronas/Documents/proj/claude-model-split.md: `diff -u` against the `.orig` copy shows only the two lines inside the fenced block changed. The block matches README.md:52-54 exactly. The alternatives paragraph was not added. Nothing else in the file changed.
- Voice: plain, short sentences, no emoji, no "simply" or "just". It matches the rest of the README.

## 2. Tests

- I ran `python3 -m unittest discover -s tests`: 88 tests, OK. That is the full suite, the same count FACTS.md records for v0.2.0.
- The spec names `python3 -m pytest tests -q`, but pytest is not installed on this Mac. The builder said so and ran the project's own unittest runner, which is the command in README.md and docs/spec.md:12. I accept that substitution. The bad command is in the spec, not the build. The architect should name the unittest command in future specs.
- `git diff --stat` lists three files: README.md, docs/spec.md and docs/tickets/FACTS.md. The FACTS.md lines were added by the scout before the build, which the scout role allows. They are facts from scout.md, not install-doc edits. The builder said this openly. Nothing in `claude-stuffer` or `tests/` was touched.
- I syntax-checked every command line from the three README blocks with `bash -n` and `zsh -n`. Both passed.
- I did not run `sudo install`, and neither did the builder, because it needs the user's password. The builder's curl + `python3 --help` dry run, as reported in build.md, is the verification the spec asked for.

## 3. Safety

- No secrets and no code changes.
- Advisory, not blocking: the recipe downloads to a fixed name, `/tmp/claude-stuffer`, and then runs `sudo install` on it. `/tmp` is world-writable with the sticky bit. On a shared multi-user machine, another local user could create that path first. Then the download fails, or that user could swap the file between `curl` and `sudo install`, and a file they control would be installed system-wide. On a single-user Mac the risk is negligible, and the spec asked for "a temp file". A hardened version would be `tmp=$(mktemp) && curl -fsSL <url> -o "$tmp" && sudo install -m 0755 "$tmp" /usr/local/bin/claude-stuffer`. If the architect wants this, it is a spec decision. The builder should not change it unasked, because the article and spec blocks must stay identical to the README block.

## 4. Would I approve the PR

Yes. The commands are correct for macOS, the no-sudo paths are right, and the scope matches the spec.
