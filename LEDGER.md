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
tool:      none (manual)
prompts:   none
review:    TODO(Gus): provider, model, the two prices, and the pricing page URL
checks:    TODO(Gus): python -m agent check-setup -> "All checks passed."
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
           TODO(Gus): your own read of the diff, what you changed or rejected
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
           TODO(Gus): your own review
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
           TODO(Gus): your own review; be ready to explain the loop in class
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
           TODO(Gus): your own review
checks:    pytest tests/test_reflexion.py: 5 passed
evidence:  REQ-012..014; HANDOUT Part 4
risk:      lesson quality depends on the model; read NOTES.md after runB

## Entry 6
artifact:  server.py at commit fa42741
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-server.md
review:    assistant: registers agent.tools.TOOLS unchanged, so descriptions are the
           docstrings; mcp.run only under __main__.
           TODO(Gus): your own review
checks:    pytest: 40 passed; python -m agent check-mcp: all checks passed
evidence:  REQ-015, ADR-005; HANDOUT Part 5
risk:      TODO(Gus): re-run check-mcp on your machine

## Entry 7
artifact:  .claude/agents/reviewer.md and .claude/agents/fixer.md at commit 54fc3d8
tool:      Claude (claude.ai agent session), model claude-opus-5-5, 2026-11-03
prompts:   session prompt; prompts/claude-agents.md
review:    assistant: both files copied from HANDOUT Part 7 as given; the fixer is
           kept unchanged so the comparison with my agent is fair.
           TODO(Gus): your own review
checks:    TODO(Gus): /agents in Claude Code lists reviewer and fixer
evidence:  REQ-016, ADR-007; HANDOUT Part 7
risk:      the reviewer's tools line is a lock only in Claude Code
```

<!--
TODO(Gus): add, in order, once each is done and written from its own files:
  Entry 8  runA experiment (from runs/runA/trace.jsonl): dataset, result, changed
  Entry 9  runB experiment (from runs/runB/trace.jsonl and NOTES.md)
  Entry 10 fixer session in Claude Code (from evidence/fixer.md, runs/vendor/judge.txt)
  Save each prompt you type (e.g. the runA rejection reason, the Part 7 prompts) in prompts/.
-->
