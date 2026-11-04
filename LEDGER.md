# Provenance Ledger

Write one entry per reviewable change or experiment, when the work is done. Use exactly
the schema in HANDOUT.md. Put the prompts you typed into AI tools in `prompts/` and name
the file in the entry's `prompts` field.

For a run, most of the entry comes straight from its trace: what the agent did and in
what order, the commands that checked it, and the result. You add what only you know:
how you reviewed it, what you rejected, and what is still uncertain.

## Worked example (not graded; leave it here and add your entries under "My entries")

This shows the level of detail expected. The commit SHAs, dates, steps and numbers are
made up, and so is the practice run.

```
## Entry 2
artifact:  agent/loop.py, run_agent, at commit 9c3a71e
tool:      Claude Code, model Claude Sonnet 5.5, 2026-11-01
prompts:   asked for the loop body from the comment above run_agent;
           prompts/loop-01.md
review:    read every line against the comment. It returned as soon as is_stuck was
           true, which skipped the tool results for the other calls in the same turn;
           moved the stuck check after the for loop as the comment says. Rejected its
           suggestion to import a framework's message classes; kept the dicts from
           core.py
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3; the comment above run_agent, steps 4 and 5
risk:      no test of mine has three tool calls in one turn

## Entry 4
artifact:  runs/try1, a practice run, agent code at commit 4b9e2d1
tool:      my agent (python -m agent run try1), anthropic claude-haiku-5-5, 2026-11-02
prompts:   the default task and BASE_SYSTEM in agent/loop.py; prompts/try1-task.md
review:    read every event in trace.jsonl. At attempt 1, steps 4 and 6, the model
           copied old_text with the read_file line numbers, and edit_file refused both
           times. At step 7 it made an edit that changed nothing the tests check, and
           at step 8 the stuck rule stopped it. I approved the step 7 edit without
           reading it closely
checks:    python -m agent check-trace try1: complete
evidence:  HANDOUT Part 1 (tool descriptions) and Part 2 (is_stuck)
risk:      one practice run; a better description may still not stop the copying
dataset:   fixture/ at commit 4b9e2d1, task "Make the tests pass."
result:    stopped stuck after 8 model calls, 14,880 input tokens, about $0.002;
           runs/try1/trace.jsonl
changed:   wrote "copy it without the line numbers" into the read_file and edit_file
           descriptions, and read every diff before approving
```

## My entries

```
## Entry 1
artifact:  Part 0 setup: .env prices and check-setup
tool:      Claude Code, model claude-opus-5-5, 2026-11-03: built .venv on Python 3.12
           and a blank .env. Key and prices entered by me
prompts:   prompts/claude-code-session-2026-11-03.md
review:    provider openai, model gpt-5.6-luna, $0.10 in and $0.50 out per million
           tokens. TODO(Gus): the pricing page URL you found them on
checks:    python -m agent check-setup: All checks passed.
evidence:  HANDOUT Part 0
risk:      prices change; recorded on the date above

## Entry 2
artifact:  agent/tools.py at commit ffa5264 (safe_path, read_file, edit_file,
           format_report, run_tests, three tool descriptions)
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt, worked from the rule comments above each function;
           prompts/claude-tools.md
review:    assistant: checked each function against its rule comment, kept every
           error text verbatim, and edited only the bodies and the three docstrings.
           safe_path counts the workspace itself as inside. edit_file writes with
           newline="\n". Descriptions tell the model to copy text without the line
           numbers (the failure in the worked example).
           me: read these functions against their rule comments. Changed and rejected
           nothing
checks:    pytest tests/test_tools.py: 16 passed
evidence:  REQ-001..006 in .agentic-sdlc/artifacts/01-requirements.md; HANDOUT Part 1
risk:      a non-UTF-8 file makes read_file crash; the toolbox reports only that the
           tool failed. The fixture has none

## Entry 3
artifact:  agent/policy.py at commit da260ee (gate_for, budget_stop, is_stuck)
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-policy.md
review:    assistant: tried tests/x.py, ./tests/x.py, tests\x.py, src/../tests/x.py,
           tests (all deny) and testsx.py, discounts.py, None, 5 (all ask) by hand.
           An absolute /tests/x.py is not denied here; safe_path refuses it instead.
           me: read these functions against their rule comments. Changed and rejected
           nothing
checks:    pytest tests/test_policy.py: 7 passed
evidence:  REQ-007..009, ADR-002; HANDOUT Part 2
risk:      none known beyond the public and hidden tests

## Entry 4
artifact:  agent/loop.py at commit 6a50719 (build_system_prompt, run_agent)
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-loop.md
review:    assistant: followed steps 1-5 of the comment in order. The stuck check is
           collected inside the for loop and acted on after it, so every call in a
           turn gets its tool result and trace event first. The trace reason is ""
           unless a person rejected. No framework; the only model access is
           provider.next_turn. Offline smoke run (scripted model, MCP tools,
           Reflexion; not committed) passed check-trace and replay.
           me: read run_agent and can explain the loop. Changed and rejected nothing
checks:    pytest tests/test_loop.py: 9 passed
evidence:  REQ-010, REQ-011, REQ-017, ADR-001/003/006; HANDOUT Part 3
risk:      not yet run against the real API; runA is the first real check

## Entry 5
artifact:  agent/reflexion.py at commit af416b4 (judge, write_lesson, reflexion)
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-reflexion.md
review:    assistant: NOTES.md is read fresh at the start of every attempt; a lesson
           line starts on a new line if the file does not end with one; a provider
           error stops without judging.
           me: read the judge and lesson writing. Changed and rejected nothing
checks:    pytest tests/test_reflexion.py: 5 passed
evidence:  REQ-012..014; HANDOUT Part 4
risk:      lesson quality depends on the model; read NOTES.md after runB

## Entry 6
artifact:  server.py at commit fa42741
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-server.md
review:    assistant: registers agent.tools.TOOLS unchanged, so descriptions are the
           docstrings; mcp.run only under __main__.
           me: read server.py. Changed and rejected nothing
checks:    pytest: 40 passed; python -m agent check-mcp: all checks passed
evidence:  REQ-015, ADR-005; HANDOUT Part 5
risk:      check-mcp re-run on this machine with .venv: all checks passed. Nothing
           beyond the public and hidden tests is known

## Entry 7
artifact:  .claude/agents/reviewer.md and .claude/agents/fixer.md at commit 54fc3d8
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-agents.md
review:    assistant: both files copied from HANDOUT Part 7 as given; the fixer is
           kept unchanged so the comparison with my agent is fair.
           me: read both agent files. Changed and rejected nothing
checks:    both ran in Claude Code 2.1.318: evidence/review.md and evidence/fixer.md
evidence:  REQ-016, ADR-007; HANDOUT Part 7
risk:      the reviewer's tools line is a lock only in Claude Code

## Entry 8
artifact:  runs/runA, re-recorded at commit b4d6db6 (experiment)
tool:      my agent, provider openai, model gpt-5.6-luna, 2026-11-03
prompts:   task "Make the tests pass."; my rejection reason;
           prompts/runA-rejection.md
review:    the first recording (commit 0f4d63a) approved the first edit at step 3
           instead of rejecting it, which did not follow HANDOUT Part 6. Its judge
           result was 6 of 6. I re-recorded once with --overwrite to follow that step,
           not to get a better result, and I keep this run although its judge failed.
           The original stays in Git history; no trace was edited. The handout's
           "record once" rule does not clearly allow this re-record.
           I did not ask the instructor before re-recording.
           From the trace: step 4 edit rejected with my reason; step 5 was a smaller
           edit with no expiry check, approved; step 6 tests passed. My reason named no
           specific problem, and the agent dropped correct code.
           I read each diff before answering at steps 4 and 5
checks:    python -m agent check-trace runA: complete, 10 events, 8 tool calls;
           python -m agent replay runA: matches its trace
evidence:  HANDOUT Part 6; RUBRIC B1, B2, B3
risk:      rule 4 is still broken in runs/runA/workspace; no visible test covers it
dataset:   task "Make the tests pass." on fixture/, hash 8aaab9373de615a5, commit b4d6db6
result:    N=1: stopped done, 7 model calls, 8 tool calls, 12575 in and 612 out tokens,
           $0.0016; visible 4 of 4; judge FAILED 1 of 6,
           test_rule_4_code_is_rejected_after_its_last_day (runs/runA/trace.jsonl)
changed:   no agent code changed; re-recorded once to include the required rejection.
           TODO(Gus): what you would do differently as a result

## Entry 9
artifact:  runs/runB, recorded at commit bfefbe7 (experiment)
tool:      my agent over my MCP server, provider openai, model gpt-5.6-luna, 2026-11-03
prompts:   none typed to an AI tool; task "Make the tests pass."
review:    from the trace: same steps as runA, with tools from server.py. The step 3
           edit was approved; attempt 1 passed, so no lesson was written and NOTES.md
           is empty. I read the diff before approving it
checks:    python -m agent check-trace runB: complete, 9 events, 7 tool calls;
           python -m agent replay runB: matches its trace
evidence:  HANDOUT Part 6; RUBRIC B1, B2, B4
risk:      with no failed attempt, lesson writing and attempt 2 did not run with a
           real model
dataset:   task "Make the tests pass." on fixture/, hash 8aaab9373de615a5, commit bfefbe7
result:    N=1: stopped done, 1 attempt, 5 model calls, 7 tool calls, 8177 in and 469
           out tokens, $0.0011; visible 4 of 4; judge 6 of 6 (runs/runB/trace.jsonl)
changed:   nothing: no agent code changed, and the run passed on attempt 1

## Entry 10
artifact:  runs/vendor (fixer session), evidence/fixer.md, runs/vendor/judge.txt
tool:      Claude Code 2.1.318, model claude-opus-5-5, 2026-11-03
prompts:   fixer task and the resume message; prompts/claude-fixer.md
review:    run headless by another Claude Code session at my request, with a
           pre-approved tool list instead of live prompts, so no person approved each
           edit as the handout asks. The first list blocked the test command; the
           session was resumed with it allowed. The fixer added code = code.upper()
           and reported the expiry bug without fixing it.
           TODO(Gus): your own read of the diff and evidence/fixer.md
checks:    python -m agent judge vendor: visible 4 of 4 passed; judge FAILED 1 of 6,
           test_rule_4_code_is_rejected_after_its_last_day; only discounts.py changed
evidence:  HANDOUT Part 7 C; RUBRIC B7, C9
risk:      rule 4 is still broken in runs/vendor/workspace, left that way on purpose
           so the comparison stays fair

## Entry 11
artifact:  evidence/terminal_mcp.md, evidence/review.md, REPORT.md draft, .mcp.json
           check, and ledger entries 8 to 11
tool:      Claude Code, model claude-opus-5-5, 2026-11-03
prompts:   prompts/claude-code-session-2026-11-03.md
review:    assistant: ran the terminal and reviewer sessions headless, generated the
           evidence files from their raw logs, and drafted REPORT.md and these entries
           from the traces, judge.txt and agent/policy.py.
           TODO(Gus): your own read of REPORT.md, and what you changed
checks:    pytest: 40 passed; check-mcp: all checks passed; check-trace and replay
           pass for runA and runB
evidence:  HANDOUT Parts 7 and 8; RUBRIC C1 to C9
risk:      the report is AI-drafted until reviewed; the reviewer's read-only lock was
           not tested with an attempted edit
```
