---
name: spare-quota
description: Smart quota burn. Spend spare weekly plan quota on a backlog the user wrote, with a quota limit, a done-when command per task and an independent checker, then run on a schedule. Teaches the pattern in a short tutorial and adapts it to the user's own backlog and CI. Use when the user says spare quota, burn quota, use up my weekly limit, "spare-quota tutorial", "spare-quota adapt" or "spare-quota tick".
---

# spare-quota

Works in Claude Code and Codex. Built only from the tool's own features: this skill, subagents, your shell, and its scheduler. No scripts, no installs.

Rules: never spend money; never send, post, delete or publish anything; only touch the project folder and `./burn/`. If something needs a permission you were not given, stop and say so. Never guess a number: unknown means UNKNOWN, and then you do nothing.

## State: one file, `./burn/burn.json`
```json
{
  "quota": {"used_pct": 41, "as_of": "2026-11-05T14:00Z", "resets_at": "2026-11-08T00:00Z",
            "stop_at_pct": 90, "max_age_hours": 24, "tasks_budget": 10},
  "parallel": 2, "review": true,
  "tasks": [{"id": "t1", "title": "what to do", "done_when": "shell command, exit 0 = done", "status": "todo", "fails": 0}]
}
```
Append one line per action to `./burn/log.md` (UTC time, what happened, exit code).

## First run (no burn.json yet, or the user says setup or tutorial)
Say: "No spare-quota setup found here. Want the 2-minute tutorial? It is 5 short questions, then you are ready. (yes / no)". On no, stop. On yes, say in two lines what this does (works your backlog with helper agents, counts a task done only when its done-when command exits 0, stops by itself on your quota limit), then ask ONE question at a time, default in [brackets], "default" is a valid answer:
1. Quota now: weekly % used and the reset time in UTC (Claude Code: `/usage`. Codex: `/status`) [no default: ask; if they cannot say, write UNKNOWN and explain that nothing will run until they give a number].
2. Stop line and budget: stop at what % [90]; how many tasks may run before you top it up [10]; how old may the number be, in hours [24].
3. Helpers at once [2]. A second, fresh helper reviews each finished task, so no result rests on one agent's word. Keep that on [yes]? (it uses a little more quota)
4. Backlog: the tasks, each with a done-when command (a test, build or script that exits 0 only if the work is right). Or "draft": propose five read-only tasks from this folder, each with a done-when, for the user to approve. A task without a done-when can never be done: say so.
5. Schedule: show the options below and ask which one [none yet].
Then write `./burn/burn.json`, show it, and run one tick.

## Adapt it to my setup (after the tutorial, or when the user says adapt)
Goal: turn the user's real backlog and real verification into burn.json tasks. Ask ONE question at a time:
1. Where does the backlog live (a TODO file, issues, a spreadsheet, their head)? Read it and propose tasks, one per item, small enough for one helper in one go. The user approves the list.
2. How do they already verify work (test, lint, typecheck, build, a CI pipeline)? Use those very commands as each task's done_when. If a check only exists remotely in CI, use a command that reads its result (for example the CI provider's CLI printing the latest run status and exiting non-zero unless it passed). Any format of "done" works as long as a command can judge it: unit tests, a Gherkin or BDD runner, a linter, a CI result, a script that checks a written standard. If their standard has no runner yet (a checklist, Gherkin scenarios without step code), propose a small script that turns it into a command and let the user approve that script before any work starts. Never invent a weaker check just to get a green.
3. Good done_when: deterministic, fast, and exits non-zero until the work is truly done. While adapting only (never during a tick), run each done_when once: if it already passes, the task proves nothing, so ask the user to fix the check. During a tick, do not read or judge the check's contents: run it and trust its exit code.
4. Where may helpers write, which commands may they run, and what must they never touch (secrets, production, other repos)? Add the answer to the rules in your report and respect it.
Then write the tasks into burn.json and run one tick.

## Tick (when the user says "tick", or a scheduled run fires)
1. Read burn.json. Stop and report (and log) if: the reset time has passed (UNKNOWN), `as_of` is older than `max_age_hours` (UNKNOWN, ask the user for a fresh number), `used_pct` is at or above `stop_at_pct` (STOP), or `tasks_budget` is 0 (STOP).
2. Pick up to `parallel` tasks (never more than `tasks_budget`) with status `todo`, a non-empty done_when, and `fails` below 2. None left: report and stop.
3. Give each task to its own subagent (if subagents are not available, do tasks one at a time). Tell it: the task title; the exact done_when command it has to satisfy (so it knows the exact file names and output expected); work in the project root and create files with exactly the names the task or check implies; never edit the check or its inputs; do NOT touch `./burn/`; do NOT say it is done; report what changed.
4. YOU (not the subagent) run each task's done_when in your shell and read the real exit code. Exit 0: status `done`, unless `review` is true (the default): then first start a fresh helper that sees only the task title, its done_when and the changed files (not the worker's report) and answers PASS or FAIL with one reason; FAIL sets status `review_failed` (it is not done and not retried until the user looks). Otherwise on a non-zero exit `fails` + 1 (at 2 the status becomes `blocked`). Subtract one from `tasks_budget` per task attempted, and append one log line per task attempted (so two tasks make two lines). Never mark a task done any other way and never edit a done_when to make it pass.
5. Report in plain words. First line: `NOT VERIFIED n of m` (tasks not yet passed). Then the quota line (used %, stop at, reading age), then what ran and what is next.

## Schedule it (all built in)
- Claude Code, in this session: `/loop 1h /spare-quota tick` (the session must stay open).
- Claude Code, away from the keyboard: Desktop app > Schedule > New local task, folder = this project, prompt `/spare-quota tick`; click Run now once and approve the prompts. It runs only while the app is open and the computer awake. (Cloud scheduled tasks start from a fresh clone and cannot see local files.)
- Codex: have cron (Mac/Linux) or Task Scheduler (Windows) run hourly: `cd /path/to/project && codex exec -s workspace-write "Use the spare-quota skill: run one tick."`
- The quota number is typed by the user, so refresh `quota.used_pct` and `as_of` when you can; when it is older than `max_age_hours` the skill stops by design.
