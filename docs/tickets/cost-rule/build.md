## mid-level build

Verdict: PASS. Edited claude-stuffer (VERSION 0.2.1, Cost sub-bullet in RULES), README.md (one sentence), and the article (4 spots).

```
Ran 90 tests in 4.530s
OK

claude-stuffer 0.2.1

 README.md             |  2 +-
 claude-stuffer        | 11 ++++++++++-
 docs/tickets/LOG.md   |  1 +
 tests/test_stuffer.py | 25 +++++++++++++++++++++++--
 4 files changed, 35 insertions(+), 4 deletions(-)
```

Note: diff --stat also lists tests/test_stuffer.py (qa) and docs/tickets/LOG.md (architect); I did not touch them.

Article diff -u:
```diff
--- /private/tmp/claude-501/-Users-kpatronas/cabbbfab-cd48-49d7-b263-74b550c333ce/scratchpad/before.md	2026-10-10 01:27:12
+++ /Users/kpatronas/Documents/proj/claude-model-split.md	2026-10-10 01:27:12
@@ -186,6 +186,10 @@
 The ladder is sequential, the team is not. On a real team several people work at once, and the architect here does the same: it can launch several subagents in one turn and gets each report as it lands. Run in parallel anything that is independent: two mids on unrelated bugs, QA writing the next ticket's tests while the senior builds this one, the reviewer and a security reviewer on the same diff. Both of those last two are read-only, so they cannot collide.
 
 Two things never run in parallel. Ladder steps: QA before the builder, the builder before the reviewer, each escalation after the previous report, because the next step's input is the previous step's output. And two writers on overlapping files: subagents share the working tree, so either serialize them or give each its own git worktree and merge afterwards.
+
+Parallel saves time, not tokens. A subagent costs the same alone or beside another, because the cost is per agent, not per minute, and the agents do not pay for talking to each other: a handoff is one prompt in and one line back, and the reports go through files. What each agent pays for is its own reading. Every tool call is a new request that carries the whole transcript so far, so a scout that answers eleven questions over 147 tool calls costs about 300k tokens whether or not a junior is working next to it. The main session is a transcript too, and the longest one of the day paid for its full length on every turn.
+
+The same arithmetic says when not to use the ladder. Each handoff is a cold start that re-reads the spec and the files, so a ticket that passes through the scout, QA, the builder and the reviewer pays for four readings of what one session would read once. On a feature that is cheap insurance. On a one-file fix it is most of the bill, and slower too. If the spec would take longer to write than the change, give it to one junior without QA or review, or make the change in place.
 
 The architect does not write code. Its value is in the design, the spec and the review. If it is typing implementation, something below it should have been asked instead.
 
@@ -240,6 +244,8 @@
 - **Batch chores.** Three renames in one project are one junior ticket, not three. Handoff overhead is per ticket, so size the ticket to the overhead.
 - **A project manager on the bench.** `pm`, Sonnet, keeps the backlog and the ticket log, grooms items until they are ticket-ready (what, where, done), writes a one-screen status on request, and chases what is blocked on whom. No code, no priority changes, closes nothing without a reviewer PASS or your word. It takes the bookkeeping off the architect, which is the cheapest work on the architect's plate and the easiest to forget.
 - **Facts outlive tickets.** The scout reads `FACTS.md` before searching and appends what it learns. The tenth ticket in a project should cost less research than the first.
+- **Narrow scout questions.** One scout per ticket with three or four questions, not one sweep that answers eleven. A broad sweep reads most of the codebase and then re-sends all of it on every later step. The next ticket's scout starts from `FACTS.md` anyway.
+- **Compact between tickets.** The main session re-reads its whole conversation every turn. Once a ticket is closed, `/compact`. The next task starts from the ticket folder, which is the point of keeping handoffs in files.
 - **The log makes patterns visible.** When every junior ticket in `LOG.md` shows an escalation, the tickets aren't junior-sized. When the senior keeps escalating to the architect, the specs are leaving too much open. You don't have to remember this; the file shows it.
 
 ## Pros and cons
@@ -265,7 +271,7 @@
 
 ### When to use it
 
-Use it when you spend real money on Claude Code, your projects have tests, and most of your requests are concrete changes. Skip it, or run only the architect plus the senior, when you mostly explore or work in codebases without tests. Trying it for a week and removing it costs nothing: the tool never deletes anything, and undoing it is deleting the files it told you it wrote.
+Use it when you spend real money on Claude Code, your projects have tests, and most of your requests are concrete changes. Even then, size the process to the task: the ladder is for features, and a chore smaller than its spec is cheaper done by one junior or in place. Skip it, or run only the architect plus the senior, when you mostly explore or work in codebases without tests. Trying it for a week and removing it costs nothing: the tool never deletes anything, and undoing it is deleting the files it told you it wrote.
 
 ## Checking it works
 
```
