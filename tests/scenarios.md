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
