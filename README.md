# review-budget

A light skill for Claude Code, Codex and Opencode that helps use LLM agents
for coding fast and without waste. It sets how many reviews, fix rounds and
test runs the work gets, and which models do them.

[Русская версия](README.RU.md)

## Why

When an assistant runs a plan with agents, most of the time goes to waiting.
A task gets a review, a fix, a re-review of the fix and another fix. The full
test suite runs after each step, the release build after each task. The
re-reviews rarely find what the first review missed. The skill comes from a
real project where these loops slowed the work more than they improved it.

## What it solves

- one review per task instead of a loop of re-reviews
- the full test suite once per task instead of after every step
- the slow release build once, at the end
- independent tasks run in parallel
- the user chooses which models the reviewers and the implementers get, so
  nobody pays for the strongest model without deciding to

The skill is self-contained: it needs no other skill or plugin.

## Algorithm

The coordinator is the assistant session that dispatches the agents.

1. **Before the first dispatch:**
   - ask once which models the reviewers and the implementers get; skip
     the question when the user's instructions already name them;
   - write a dependency table: the files and interfaces of each task;
   - start a warm test environment, if the project allows one.
2. **Each task:**
   - the implementer writes the code and runs only the tests it touches;
   - the coordinator runs the full suite once;
   - one review on the reviewer model, which reads that test output;
   - one fix round with every finding.
3. **No re-review of the fix**, unless it changes an interface other code
   uses or touches a file no finding named.
4. **Tasks with no shared files or interfaces** run at the same time, each in
   its own worktree.
5. **At the end:** one whole-branch review, one fix round, the full suite,
   then the release build once.
6. **Docs, comments, version bumps and one-line fixes** are done by the
   coordinator itself.
7. **A small change after the run** is made by the coordinator, with one
   review and one build — no new plan.

## Install

```bash
git clone https://github.com/vadbosh/review-budget
cd review-budget
./install.sh            # every assistant found under ~/.claude, ~/.codex, ~/.config/opencode
./install.sh --dry-run  # show what would be written
./install.sh --no-rule  # the skill only, without the trigger rule
```

The installer also adds a trigger rule, `rules/review-budget-trigger.md`. It
tells the assistant to load the skill whenever work goes through agents, also
when another process skill was loaded first. Without the rule, say
`/review-budget` (`$review-budget` in Codex) before a run.

## Models by assistant

| Assistant | Question | Per-agent model |
|---|---|---|
| Claude Code | `AskUserQuestion` | `model` on each `Agent` call |
| Codex | `request_user_input` | `model` on each `spawn_agent` call |
| Opencode | `question` | `model` on each `subagent` call (Opencode 2.x) |

MIT License.
