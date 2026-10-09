
## mid

Verdict: done. Edits as specified in claude-stuffer, README.md line 26, and the article (three spots).

```
----------------------------------------------------------------------
Ran 92 tests in 4.769s

OK
claude-stuffer 0.2.2
 README.md             |  2 +-
 claude-stuffer        | 20 ++++++++++++++------
 tests/test_stuffer.py | 16 ++++++++++++++--
 3 files changed, 29 insertions(+), 9 deletions(-)
```

Article diff -u (cut to 200 cols):
```
--- /private/tmp/claude-501/-Users-kpatronas/cabbbfab-cd48-49d7-b263-74b550c333ce/scratchpad/article.before.md	2026-10-10 01:45:19
+++ /Users/kpatronas/Documents/proj/claude-model-split.md	2026-10-10 01:45:19
@@ -32,7 +32,7 @@
 |------|-------|-------|------|
 | Reviewer | Senior | Opus, no code edits | Reads a finished change against its spec, runs the tests itself, returns PASS or FAIL with reasons tied to files and lines. Writes only its review. |
 | QA engineer | Mid | Sonnet | Writes the tests from the spec before anyone builds, runs them, confirms they fail for the right reason. Never writes the implementation. |
-| Scout | Junior | Haiku, no code edits | Answers the architect's questions about the codebase: where is this used, what calls that, which tests cover it. Facts with file paths and line numbers. Writ
+| Scout | Junior | Haiku, no code edits | Looks things up for the architect: which files a change touches, which tests cover them, where a symbol is used. Facts with file paths and line numbers, at m
 | Project manager | Mid | Sonnet, no code | Keeps the backlog and the ticket log, grooms items until they are ticket-ready, writes status. Never sets priorities. |
 
 Why those levels: a reviewer weaker than the author misses exactly the bugs you care about, so the reviewer is Opus. QA sits one level below the builder because turning a good spec into tests is well
@@ -172,7 +172,7 @@
 The example shows the general shape:
 
 1. You talk to the architect. That's the normal Claude Code session.
-2. The architect asks the scout what it needs to know, then writes a spec. A spec has four parts, and if any is missing the task is not ready: the files to touch, the constraints, what "done" means, 
+2. The architect asks the scout for the lookups the spec needs (files, tests, callers), then writes it. A spec has four parts, and if any is missing the task is not ready: the files to touch, the con
 3. QA writes the tests from the spec. They fail, which is the point.
 4. The architect hands the spec and the tests to the right level or role. A subagent starts with no memory of your conversation, so the spec has to be complete on its own.
 5. The builder does the work, runs the tests, reports.
@@ -244,7 +244,7 @@
 - **Batch chores.** Three renames in one project are one junior ticket, not three. Handoff overhead is per ticket, so size the ticket to the overhead.
 - **A project manager on the bench.** `pm`, Sonnet, keeps the backlog and the ticket log, grooms items until they are ticket-ready (what, where, done), writes a one-screen status on request, and chas
 - **Facts outlive tickets.** The scout reads `FACTS.md` before searching and appends what it learns. The tenth ticket in a project should cost less research than the first.
-- **Narrow scout questions.** One scout per ticket with three or four questions, not one sweep that answers eleven. A broad sweep reads most of the codebase and then re-sends all of it on every later
+- **Scouts look up, they do not survey.** The first version of this setup let the scout answer "how does the camp state machine work". One such scout ran 214 steps to a 300k-token transcript, re-read
 - **Compact between tickets.** The main session re-reads its whole conversation every turn. Once a ticket is closed, `/compact`. The next task starts from the ticket folder, which is the point of kee
 - **The log makes patterns visible.** When every junior ticket in `LOG.md` shows an escalation, the tickets aren't junior-sized. When the senior keeps escalating to the architect, the specs are leavi
 
```
