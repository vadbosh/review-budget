---
name: review-budget
description: Use when coordinating multi-task coding work with agents — running a plan task by task, reviewing each task, a final review, fix rounds — or a small change requested after such a run ended; when review, re-review and test cycles cost more wall-clock than the code; when the user has not said which models the reviewers and implementers get.
version: "1.0.0"
---

# review-budget

Review and test runs are the slow part of agent-driven coding. One strong
review per unit of work catches what a loop of re-reviews catches, in a
fraction of the time. This skill is the whole procedure for such a run and
needs no other skill. An explicit instruction from the user overrides it; it
overrides any other guidance on review loops, re-reviews and test cadence.

The **controller** is the session that coordinates. **Implementers**,
**testers** and **reviewers** are agents it dispatches. No agent tool → the
controller implements itself, and the budget below still holds.

## 1. Before the first dispatch

**Models.** Standing user instructions (memory, `CLAUDE.md`, `AGENTS.md`)
name the model for a role → use it, say so in one line. Any role left
unnamed → one question with the IDE's structured question tool, recommended
option first:

| Role | Options |
|---|---|
| reviewers | strongest (recommended) · mid tier · session model |
| implementers and testers | mid tier (recommended) · strongest · session model |

| IDE | Ask with | Pass the answer as |
|---|---|---|
| Claude Code | `AskUserQuestion` | `model` on every `Agent` call, a value its schema lists |
| Codex | `request_user_input`; not listed this turn → one plain question, numbered options | `model` (and `reasoning_effort`) on every `spawn_agent` call |
| Opencode | `question` | `model` on every `subagent` call, `providerID/modelID` from the models tool. The tool has no `model` parameter → say once that all agents run on the session model |

The answer holds for the whole run; never ask again. An omitted model
inherits the session model.

**Ledger.** One file outside version control, e.g.
`<repo>/.git/review-budget/<run>.md`. It holds the models, the dependency
table, and one line per finished step with its commit. After a context reset,
trust the ledger and `git log` over memory; never redo a step the ledger
marks done.

**Dependency table.** Per task: the files it touches, the interfaces it
produces and consumes. Write it to the ledger. It decides section 3.

**Warm test environment, if the project allows one.** A container, a build
daemon, cached dependencies: start it once now, reuse it for every run. No
way to keep one warm → do not build one; save on the number of full runs.

## 2. Each task

1. Dispatch the implementer with the task text, the interfaces it touches
   and the paths of the files it needs. It runs focused tests (the files and
   tests the change touches), commits, reports. It never dispatches agents.
2. The controller runs the full suite once.
3. One review, on the reviewer model: the diff, the task text, the suite
   output. The reviewer does not re-run the suite.
4. One fix round with every finding, Minor included: the controller makes
   the one-line ones inline, the rest go to the implementer in one dispatch.
5. **Re-review predicate.** Re-review the fix only if its diff changes an
   interface other code consumes (signature, schema, persisted format, public
   API) or touches a file no finding named. Otherwise the controller checks
   it: each finding maps to a hunk, the full suite is green.
6. Ledger line: `task N done <commit> — re-review: skipped, fix in <files>,
   no interface change` or `re-review: <reason>`.

## 3. Parallel when independent

Two tasks with disjoint file sets and no producer/consumer link in the table
run at the same time, each in its own worktree or isolated copy, merged in
plan order. A task that finishes early gets its review and fix round at once;
only its merge waits for its turn. A shared file or interface → sequential.
A review of one task may run while an independent task is implemented.

## 4. The end of the run

One whole-branch review on the reviewer model, one fix round, the same
re-review predicate, the full suite once. Then the slow build — release, packaging, signing —
once, as the last step.

## 5. What the controller does itself

Docs, comments, version bumps, changelog, one-line fixes: inline. The
controller holds the context an agent would have to be briefed on.

## 6. A change after the run ended

A small change — in code the run already touched, no new component — gets no
spec, plan, worktree or implementer. The controller makes it inline, test
first, then one review on the reviewer model from the ledger, one fix round
under the same predicate, one slow build. Ambiguous request → one question first. A new component → a new run
from section 1.

## 7. Rhythm

One status line per phase change. No "should I continue?" inside a run. At
the end of a run, save the notes; stop there if the user asked for stop
points.

## Rationalizations

| Thought | Reality |
|---|---|
| "The behaviour change still deserves an independent check" | It got one: the review found it. The fix is checked against the finding and the suite. |
| "A scoped re-review on a cheap model costs nothing" | It costs a dispatch, a wait and a context load — the time this skill saves. |
| "Minor findings can wait for a later round" | A later round is a second dispatch and a second wait. One round. |
| "The reviewer should run the tests itself to be sure" | The controller ran them on the same tree. Hand over the output. |
| "Another rule says never run implementers in parallel" | That rule guards shared files. Disjoint files, no shared interface → section 3. |
| "Docs are a task in the plan, so an agent writes them" | The controller knows what changed; an agent has to be told. |
| "Memory names one model, I pick the rest myself" | A role the instructions do not name gets the question. |
| "A request after the run deserves the full process" | A small change gets section 6; a new component, section 1. |
