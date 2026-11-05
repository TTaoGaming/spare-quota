# spare-quota: a seed for burning your spare AI quota on useful work

Flat-rate AI plans reset every week, and quota you do not use is wasted. This is a small pattern, plus one agent skill, for getting your money's worth: give your agents a quota limit and a backlog with a checkable definition of done, and let an orchestrator keep dispatching helper agents until the work passes or the quota line is reached. It replaces clicking through every thread yourself.

Markdown only. No code, no installs: it uses what your agent already has (skills, subagents, a shell, a scheduler).

## Use it: point your agent here
Paste this to Claude Code or Codex, in the project you want to work on:

> Read the README.md and spare-quota/SKILL.md in this repo: https://github.com/TTaoGaming/spare-quota. Install the skill into my project (.claude/skills/ for Claude Code, .agents/skills/ for Codex), walk me through the tutorial, then adapt it to my own backlog and my own verification and CI.

That is the whole setup. The skill explains itself, asks 5 short questions, then adapts to your backlog and to however you already verify work (unit tests, Gherkin/BDD, lint, a CI result, a script that checks a written standard). Any definition of done works as long as a command can judge it.

## The 5 parts
1. **Quota.** You say how much of your week is used and when it resets. It stops at your limit and stops if the number is old. It never guesses.
2. **Backlog.** Small tasks, one per item.
3. **Definition of done.** Each task has a command that exits 0 only when the work is right.
4. **Subagent dispatch.** The orchestrator hands each task to a helper and keeps going until the backlog passes, the budget is spent, or the quota line is hit.
5. **Independent checkers.** A helper never marks its own work done: the orchestrator runs the done command itself. By default a fresh second helper also reviews each result without seeing the first one's report. A task that fails twice is blocked, and the agent may not edit a done command to make it pass.

## What we tested, and where
Seven small cases, scored by a script that only looks at files, so an agent saying "done" counts for nothing: normal two-task run, over the stop line, stale quota number, spent budget, a check that can never pass, a bug fix where the test file must stay untouched, and the tutorial.
- **Claude Code 2.1.318 (Linux):** all 7 pass on the final skill, and the impossible-check case 3 of 3. The main case passed 5 of 5 earlier, and 3 of 3 with the independent checker on.
- **Codex CLI 0.165.0 (Windows):** all 7 pass on an earlier version of the skill, and the main case passed 2 of 2 with the independent checker on, with the tutorial and stop-line cases also passing on a near-final version. The run on the exact final version was still going when this was written; see [EVIDENCE.md](EVIDENCE.md).
- Run details and the one real bug we found and fixed are in [EVIDENCE.md](EVIDENCE.md).

## Weak points (read these)
- **The checker is the same model family as the worker.** A same-family reviewer can share blind spots. It is on by default; real independence means a different tool, or you.
- **The quota is typed by you, not read live.** Forget to refresh it and the skill stops by design.
- **Done is only as good as your done command.** A weak check can still be gamed.
- Tested on Claude Code and Codex only. Gemini and other tools: not tested.
- The adapt-to-your-backlog mode was tried once on a small project (Claude Code only).
- No A/B test yet that this beats you managing quota by hand. Whether unused quota expires at your reset is unknown: check your plan.
- Scheduled runs (loop, Desktop task, cron), many helpers at once and long unattended runs are not tested. Samples are small.

Found a problem? Tell us what broke.
