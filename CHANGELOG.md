# Changelog

Versions are the `version:` field in `skills/review-budget/SKILL.md`, and each
is tagged at the commit that introduced it.

Releasing, in one commit: bump `version:`, add the section here, commit, then
`./release.sh tag` and `git push --tags origin`. A tag is not moved afterwards.

## 1.0.1 — 2026-11-08

- **Works next to a process skill that has its own rules for models and its
  own ledger.** The user's answer about models now outranks any other rule
  that picks a model per task, per size or per round. When another process
  already keeps a ledger for the run, the skill writes its lines there: one
  run, one ledger.

## 1.0.0 — 2026-11-08

- **First release.** One review per task, one fix round with every finding,
  no re-review of a fix unless it changes an interface or touches a file no
  finding named. The full suite once per task, run by the coordinator; a warm
  test environment when the project allows one; the slow build once, at the
  end. Independent tasks run in parallel in their own worktrees. Docs and
  one-line fixes are done by the coordinator; a small change after the run gets
  one review and one build.
- **Models are asked once.** Reviewers and implementers get the model the user
  picks: `AskUserQuestion` and `Agent` in Claude Code, `request_user_input` and
  `spawn_agent` in Codex, `question` and `subagent` in Opencode 2.x.
- **Self-contained.** Needs no other skill or plugin.
