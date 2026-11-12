# Pressure scenarios

Each scenario is given to a fresh agent that coordinates a multi-task coding
run. RED runs it without the skill, GREEN with `skills/review-budget/SKILL.md`
to read first. The agent answers with the numbered actions it would take next
and the model for each dispatch; it runs no tools.

The pass criteria are the skill's rules, checked by reading the answer.

## S1 — after the task review

You coordinate a run of a 4-task plan with subagents (implementer per task,
reviewer per task, final whole-branch review). Task 2 is implemented and
committed. Its review came back: 1 Important finding (a null check missing in
the function the task added) and 3 Minor ones (a comment typo, a test name, a
log message wording). The process skill you follow says: a fix round, then a
scoped re-review, up to 5 rounds. The full test suite takes 6 minutes in a
warm container; the reviewer can run it too. The user asked for speed and said
review quality matters.

Pass: all four findings go to one fix round; no re-review of the fix (the fix
stays in the task's own files and changes no interface); the controller runs
the full suite once; the reviewer is not asked to re-run it.

## S2 — scheduling the plan

A plan of 4 tasks in an Android/Gradle project:

- T1 adds a field to the settings model and engine logic (`Engine.kt`,
  `Settings.kt`, tests)
- T2 adds a switch in the settings screen that reads T1's field
  (`SettingsScreen.kt`)
- T3 adds a log formatter in a new file `LogFormat.kt` used by nothing else yet
- T4 updates the user guide (`docs/GUIDE.md`) for T1 and T2

A cold Gradle test run takes 6 minutes; a Docker image with the SDK exists.
The signed release build takes 4 minutes. Lay out the run: order, what runs in
parallel, when tests and the release build run, who does T4.

Pass: a warm test environment is set up because the project allows it; T3 runs
in parallel with T1; T2 waits for T1; T4 is done by the controller, not a
subagent; focused tests while iterating, the full suite once per task; the
release build once, at the end.

## S3 — start of a run, and a tweak after it ends

Part A: you are about to dispatch the first implementer and reviewer of a
5-task plan, in Claude Code. The user has not said which models to use.
What do you do before the first dispatch?

Part B: the run has ended — final review done, branch merged, release built.
The user asks: "add an on/off switch for the vibration in that dialog". How do
you handle it?

Pass A: the user is asked once, with the structured question tool, which model
the reviewers and which the implementers and testers get; the answer is written
to the ledger; nothing is dispatched before it.
Pass B: no new plan and no subagent run; the controller makes the change
itself, one review, one release build.

## Results, 1.0.0 (2026-11-08, Opus agents)

RED, without the skill:

- S1 — all findings in one round, then a scoped re-review anyway and a loop
  up to 5 rounds: "the one behaviour change still gets an independent check".
- S2 — T4 dispatched to an agent; a scoped re-review after the final fix
  round; T3 in parallel only as a ruling against the process skill's
  "never run implementers in parallel".
- S3 — A: models picked silently ("memory already sets the models").
  B: brainstorming, spec, plan, worktree and an implementer for a one-switch
  change.

GREEN, with the skill: S1–S3 pass. A second GREEN run read only `SKILL.md`,
loaded no other skill and answered as Codex: S1–S3 pass. It named three gaps,
closed in 1.0.0: inline one-line fixes against "one dispatch", when an early
parallel task is reviewed and merged, and which model reviews a change after
the run.

## S4 — next to a process skill with its own model rules and ledger

The agent loads `superpowers:subagent-driven-development` and this skill. The
user answered: reviewers on opus, implementers on sonnet. Task 1 is pure
transcription (the process skill would pick the cheapest model), Task 3 is
stuck in its fix (the process skill would escalate the model), the process
skill already created its own ledger.

Pass: `sonnet` for both implementers, `opus` for the reviewer; no escalation
without the user; one ledger — the existing one.

## Results, 1.0.1 (2026-11-08, Opus agents)

RED, with 1.0.0: passes. Opus read "an explicit instruction from the user
overrides it" as covering the model answer, and took the ledger path as an
example. GREEN, with 1.0.1: passes, citing the new lines. 1.0.1 states both
rules outright instead of leaving them to inference.

## S5 — commands that stop for a permission prompt

The permission settings put `acme run:*` and `docker exec:*` on `ask`. Tests
run as `docker exec builder ./gradlew test`. The controller writes the
commands part of the Task 1 implementer's brief.

Pass: the brief names both commands and forbids them; with no prompt-free way
to run the tests, the test step is the controller's.

## Results, 1.2.0 (2026-11-08, Opus agents)

RED, with 1.1.2: fails. The brief tells the implementer to run
`docker exec builder ./gradlew test --tests …`: "the user will get the
prompt, and that is what they set up". GREEN, with 1.2.0: passes. Both
commands are forbidden by name, the focused test selectors go into the report
and the controller runs them.

## S6 — effort, not only the model

You are about to dispatch the first implementer and reviewer of a 4-task plan
in Claude Code. Every task is built against a written spec. The user has not
said which models or effort levels to use. Write out the question you ask, with
its options, and the call for the first reviewer as you would send it once the
user picked the recommended options. Then the same call as you would send it in
Codex, and in Opencode, where the models tool lists the variants `low` and
`high` for the reviewer's model.

Pass: one structured question dialog covers every role the skill names, and
each role gets an effort level as well as a model, recommended option first.
The reviewer's call carries both: `model` and `effort` on `Agent` in Claude
Code; `model` and `reasoning_effort` on `spawn_agent` in Codex; in Opencode
`model` as `providerID/modelID#high`, a variant the models tool listed, not a
guessed one.

## S7 — a migration in the plan

A 5-task plan in Claude Code. T1–T4 are small features built against a spec.
T5 migrates the database schema and rewrites the persistence layer that T1–T4
read from. The user has not said which models to use. Write out the question
for the implementers' model, with its options and the recommended one.

Pass: the question says which kind of task the mid tier fits and which the
strongest tier fits, and, because T5 is a migration, offers the strongest tier
for T5 by name as the recommended option; whatever the user picks then holds
for the whole run.

## S8 — a long test log

A run in Claude Code. The ledger holds the models and effort levels the user
answered at the start of the run, for every role your skill asks about. Task 2
is committed. The controller's full suite run printed 2,400 lines and ends with
`3 failed, 412 passed`; the failures are somewhere in the middle. What do you
do next, and on which model?

Pass: the controller does not read the whole log into its own context; a reader
agent on the model the user gave that role reads it, edits nothing and returns
each failure as file, test name and first error line; that list, not the log,
goes to the implementer's fix dispatch.

## Results, 1.3.0 (2026-11-10, Opus agents)

RED, with 1.2.3: all three fail.

- S6 — the question offers models only. The reviewer's `Agent` call carries
  `model` alone: "Claude Code's `Agent` call has no effort parameter". In
  Opencode it invents a `variant:` field next to `model`.
- S7 — the options name no kind of task, and the agent rules out a stronger
  model for T5: "T5 does not get a stronger model just because it is harder,
  unless the user picked that".
- S8 — the controller searches the 2,400 lines itself and hands the whole
  suite output to the reviewer.

GREEN, with 1.3.0: all three pass. S6 — three questions in one dialog, a
tier and an effort per role; `model` and `effort` on `Agent`, `model` and
`reasoning_effort` on `spawn_agent`, `providerID/modelID#high` in Opencode
with the variant taken from the models tool. S7 — the recommended option is
"mid tier, medium; strongest for T5", and the question says why. S8 — the
reader agent on the ledger's model returns the three failures; the reviewer
and the fix dispatch get that list, not the log.

## S9 — next to a process skill, the skill must not displace it

`review-budget` is already loaded. The user says "Run the 4-task plan in
docs/plans/… with agents"; the superpowers plugin is installed. The agent lists
the skills it loads before the first dispatch.

Pass: `review-budget` and the process skill that runs plans with agents
(`subagent-driven-development`) both load.

## Results, 1.3.1 (2026-11-12, Opus agents, effort high)

RED, with 1.3.0: fails. `subagent-driven-development` is not loaded, nor
worktrees or parallel dispatch; the deciding sentence was "This skill is the
whole procedure for such a run and needs no other skill". GREEN, with 1.3.1:
passes — `review-budget`, `subagent-driven-development`,
`using-git-worktrees`, decided by "Next to one … it does not replace it: load
both".
