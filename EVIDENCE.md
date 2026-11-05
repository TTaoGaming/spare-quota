# Evidence: what was run, on what, and what broke

Cases (scored by a script that reads files on disk; the agent's own report counts for nothing):
C1 normal two-task run; C2 over the stop line; C3 stale quota number; C4 spent task budget; C5 a done command that can never pass (two rounds, must end blocked); C6 fix a bug without editing the test; C7 tutorial from scratch with scripted answers.
Skill versions are identified by the first 8 characters of the SHA-256 of `spare-quota/SKILL.md`. The final one is `0a28377d`. The test scripts are not in this repo (it is Markdown only); ask for them if you want to rerun.

## Claude Code 2.1.318, Linux
| Skill version | What ran | Result |
|---|---|---|
| f6615328 (first) | all 7 cases | 7/7 (after we fixed a quoting bug in our own test script) |
| f6615328 | main case C1, 5 repeats | 3/5: two runs wrote one combined log line for two tasks |
| f6615328 | an independent reviewer session, all 7, then C1 twice more | 6/7; C1 passed 1 of 3: one run a helper wrote a wrongly named file |
| 52f14beb | C1, 5 repeats; all 7 | 5/5; 7/7. Fix: helpers are told the exact check and to create files by exact name; one log line per task |
| 52f14beb + checker on | C1 with the independent checker enabled, 3 repeats | 3/3 |
| 1a4bbc06 | all 7 | 6/7: C5 failed. A line meant for "adapt" mode leaked into normal rounds, so the agent read the check script and refused to dispatch a task whose check can never pass. Fixed by limiting that advice to adapt mode |
| 0a28377d (final) | all 7; C5 three repeats | 7/7; C5 3/3 |
| 1a4bbc06 | adapt mode on a tiny project (TODO.md and a test file), one run | Passed: it ran both checks first (both failed, as they should), dispatched two helpers, checker passed, test file untouched. It also noted the two checks depended on each other. n=1 |

## Codex CLI 0.165.0, Windows
| Skill version | What ran | Result |
|---|---|---|
| f6615328 | all 7 | C2 to C7 pass. C1: the agent completed both tasks with two real helper agents and ran the checks itself, but our test script rejected the file because Windows PowerShell saved it with a byte-order mark. We fixed the test script and rescored C1 as pass |
| 52f14beb | all 7 | 7/7 |
| 1a4bbc06 | C1 with the checker on (2 runs), C7, C2 | all pass |
| 0a28377d (final) | in progress when this was written (C1 with the checker on, C5, C6, C7); results to be added | |

## What we fixed because of testing
- The skill's main case flaked until helpers were given the exact check and file names.
- An "adapt" instruction changed the behavior of normal rounds until we scoped it.
- Two bugs in our own test script (shell quoting; Windows file encoding).

## Not tested
Scheduled runs, more than two helpers, long unattended runs, Gemini and other tools, a different-family checker, and any A/B comparison against managing quota by hand.
