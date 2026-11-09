# review-budget Auto-Trigger

Work done through agents — running a plan task by task, dispatching
implementers or reviewers, fix rounds, a final review — or a small change right
after such a run → invoke the `review-budget` skill BEFORE the first dispatch
(`$review-budget` in Codex). Another process skill for the run already loaded →
load `review-budget` next to it.

Not for: one edit with no agents and no review.

Why: the skill sets one review per task, when a fix is re-reviewed and which
models do the work; a run started without it repeats reviews and test runs.
