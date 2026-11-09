# Changelog

Versions are the `version:` field in `skills/review-budget/SKILL.md`, and each
is tagged at the commit that introduced it.

Releasing, in one commit: bump `version:`, add the section here, commit, then
`./release.sh tag` and `git push --tags origin`. A tag is not moved afterwards.

## 1.2.2 — 2026-11-09

- **`release.sh check` fails when a shipped file is not in HEAD.** Every other
  check reads the working tree, so the uncommitted trigger rule fixed in 1.2.1
  passed as `agreed and released`. The skill files, the rule, `lib/wire.py`
  and `install.sh` are now looked up in HEAD — what a clone actually gets.

## 1.2.1 — 2026-11-09

- **The trigger rule is in the repository.** `rules/review-budget-trigger.md`
  matched the `.gitignore` pattern `review-*.md`, meant for cold-review files at
  the root, and was never committed. A fresh clone installed the skill and
  failed to wire the rule in all three IDEs with `No such file or directory`.
  The pattern is anchored to the root now: `/review-*.md`.

## 1.2.0 — 2026-11-08

- **Agents never wait on a permission prompt.** Every brief names the
  commands the IDE's permission settings put on `ask` and forbids them, with
  the command to use instead; where there is none, the controller runs that
  step itself. One such prompt per agent call had held whole runs until a
  human answered.

## 1.1.2 — 2026-11-08

- **The README names the repository to clone**, now that it is published at
  `github.com/vadbosh/review-budget`.

## 1.1.1 — 2026-11-08

- **`release.sh check` reads shipped files whose path has a space.** The leak
  check passed the file list word-split; a `$HOME` with a space in it was read
  as two paths that do not exist. The list is an array now.

## 1.1.0 — 2026-11-08

- **A trigger rule loads the skill whenever work goes through agents.**
  `rules/review-budget-trigger.md` goes to `~/.claude/rules`,
  `~/.config/opencode/instructions` (with an entry in `opencode.json`
  `instructions[]`) and `~/.codex/memories` (with an `@`-reference in
  `~/.codex/AGENTS.md`). Before, the skill loaded only by its description,
  which was checked once and in Claude Code only. `./install.sh --no-rule`
  installs the skill alone; `./release.sh check` compares the rule copies too.

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
