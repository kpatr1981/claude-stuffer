# Ticket log

| date | slug | built by | escalations | verdicts |
|---|---|---|---|---|
| 2026-10-10 | claude-stuffer-0-1-0 | senior (Opus); tests by qa (Sonnet) | none on the ladder; reviewer FAIL twice (EOF confirmation bypass; dangling symlink write-through), both fixed by senior | reviewer FAIL, FAIL, then fixes verified by architect; released as v0.1.0 |
| 2026-10-10 | claude-stuffer-0-2-0 | senior (Opus); tests by qa (Sonnet) | reviewer FAIL twice (embedded bench carried a stale marker line; slug/role fullmatch; symlinked docs/tickets), fixed by senior | reviewer PASS on third pass; released as v0.2.0 |
| 2026-10-10 | macos-install | mid (Sonnet); docs only, no qa | none | reviewer PASS first pass (note: /tmp download path, non-blocking) |
| 2026-10-10 | agents-wizard-removed | junior (Haiku); docs plus one string and its test, no qa/reviewer | none | junior DONE, 88 tests OK, commit ee28a04 pushed; architect re-pushed the live Ghost article |
| 2026-10-10 | cost-rule | mid (Sonnet); tests by qa (Sonnet) | none | reviewer PASS first pass; version 0.2.1; global CLAUDE.md block refreshed with `init`; article republished |
| 2026-10-10 | scout-lookup | mid (Sonnet); tests by qa (Sonnet) | none | reviewer PASS first pass; version 0.2.2; bench refreshed with `init --force`; article republished |
| 2026-10-10 | lean-agents | mid (Sonnet); tests by qa (Sonnet) | builder BLOCKED on a wrong test expectation that was the architect's (reviewer tool list); architect fixed one string | reviewer PASS first pass; version 0.2.3; bench refreshed with `init --force`; drop64 role files given tools lines by hand; article republished |
| 2026-10-10 | read-ranges | junior (Haiku); no qa/reviewer (one-sentence chore) | none | junior PASS, 96 tests OK; version 0.2.4; bench refreshed; drop64 role files given the output paragraph by hand |
| 2026-10-10 | architect-opus | junior (Haiku); no qa/reviewer (string chore) | junior PARTIAL: one README phrase the spec missed; architect fixed the word | 96 tests OK; version 0.2.5; global block refreshed; article republished; settings.json model set to claude-opus-5-5 |
| 2026-10-10 | compact-rule | senior (Opus); tests by qa (Sonnet) | none | reviewer PASS first pass, 115 tests OK; version 0.2.6; context-guard hook written by `init`, snippet printed (settings.json untouched); extra `is_ours` shebang change accepted; 3 nits deferred to PENDING; global block refreshed and hook written with `init`; article republished (via Chrome, Ghost admin session) |
2026-10-10 | scout-grep-cap | junior (1 restart: worktree base was stale origin/main, architect error) | architect check PASS, 116 green | c471e7d
