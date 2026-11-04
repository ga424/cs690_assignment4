# Assignment 4 Report

Name: Gustavo Abreu
Commit graded: TODO(Gus): the final commit SHA

Answer from your own traces. Give step numbers as `attempt 1, step 4`. Keep to the word
limits; a short answer with the right evidence earns full points.

## 1. The runs

Run `python -m agent summary` and paste its table here exactly as printed.

| Run | Tools | Judge | Stopped (per attempt) | Model calls | Tool calls | Input tokens | Output tokens | Cost | Judge verdict |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| runA | local | after | 1: done | 7 | 8 | 12575 | 612 | $0.0016 | 1: FAIL |
| runB | mcp | reflexion | 1: done | 5 | 7 | 8177 | 469 | $0.0011 | 1: PASS |
| try1 | local | after | 1: done | 5 | 7 | 8032 | 463 | $0.0010 | 1: PASS |

## 2. A rejection is feedback (Run A, at most 100 words)

Which step proposed the edit you rejected, and what was your reason? What did the agent
do next, and at which step? Did it follow your reason?

I rejected the edit proposed at attempt 1, step 4. My reason was "I do not agree with the recommended agent solution and next step". At attempt 1, step 5 the agent proposed a smaller edit: case normalization only, without the expiry check. It followed the reason as far as it could, but the reason named no specific problem, so it guessed. It treated the rejection as a problem with the size of the "combined edit" and dropped correct code. This is a re-recording; ledger entry 8 explains why.

## 3. The visible tests and the judge (Run A, at most 120 words)

What did the last `run_tests` in runA report, and what did the judge say after the run?
If the judge failed, which rule in `spec.md` did the code break, at which step did the
agent stop, and why did it stop there?

The last `run_tests` (attempt 1, step 6) reported "PASSED: 4 of 4 tests passed." The judge reported "FAILED: 1 of 6 tests did not pass", on `test_rule_4_code_is_rejected_after_its_last_day`. The code broke rule 4: a code must raise `ValueError` after its last day. The agent stopped at attempt 1, step 7 by replying without a tool call, because the visible tests passed and none of them covers expiry. It never saw the judge.

## 4. The lessons (Run B, at most 150 words)

Quote every lesson runB wrote to `NOTES.md`. For each one, say whether it names the real
cause, points at evidence, and says what to do next. Did attempt 2 follow it? Give the
step that shows it. If attempt 1 passed, say which step shows why no lesson was needed.

runB wrote no lessons, and `NOTES.md` is empty, so there is nothing to quote or judge. Attempt 1 passed, so there was no attempt 2. Attempt 1, step 4 shows why no lesson was needed: after the step 3 edit, `run_tests` reported "PASSED: 4 of 4 tests passed", and the judge then reported "PASSED: 6 of 6 tests passed." Reflexion was therefore never exercised in this run.

## 5. Memory grows (Run A, at most 60 words)

Give `input_tokens` from the first and the last tool event in `runs/runA/trace.jsonl`,
with their step numbers. Why does the number grow from step to step?

First tool event: 811 `input_tokens` at attempt 1, step 1. Last: 2375 at attempt 1, step 6. It grows because every model call resends the whole conversation, including all earlier tool calls and results.

## 6. MCP (at most 120 words)

Quote one of your tool descriptions exactly as `python -m agent check-mcp` showed it in
the tools/list reply. Then name one risk of installing someone else's MCP server, and say
which part of your agent would stop it, or that nothing in your agent would.

`list_files`, as tools/list showed it:

```
"List every file in the project, one path per line, relative to the project folder.\n\n    Takes no inputs. Use it first to see what the project contains; the paths it returns\n    are the paths read_file and edit_file expect.\n    "
```

Risk: someone else's server runs on my machine with my permissions, so a tool called `read_file` could secretly upload files. Nothing in my agent would stop that. `gate_for` checks only the tool's name, so it would allow that call, and it cannot see what the server does.

## 7. The reviewer and the judge (at most 100 words)

What did your reviewer find in runB's final `discounts.py`? Did it find anything the
judge did not, or miss anything the judge caught? In the tool you used, was the
reviewer's read-only rule a lock or a request, and how do you know?

The reviewer found no violations: all five rules were followed. The judge agreed, 6 of 6. It missed nothing the judge caught. It also raised points the judge does not test, such as `"ſave10"` being accepted, though it called these unsettled by the spec. In Claude Code the read-only rule is a lock: the `tools` line lists only Read, Grep and Glob, so edit tools are unavailable. The session used only Read and Glob; I did not test it with an attempted edit.

## 8. Your agent and a vendor's agent (at most 120 words)

What did the fixer change, and what did `python -m agent judge vendor` report for the
visible tests and for the judge? Did it change anything in `tests/`? Compare this with
runA, where your own agent had the same task on the same project. Then name one thing you
could control in your own agent that the fixer file could not set, and one thing that was
easier with the vendor's agent.

The fixer added `code = code.upper()`. `judge vendor` reported visible tests 4 of 4 passed, but the judge failed 1 of 6: `test_rule_4_code_is_rejected_after_its_last_day`. It changed nothing in `tests/`. runA ended the same way: 4 of 4 visible, the same judge failure. runA proposed the expiry check at step 4 but dropped it after my rejection; the fixer noticed the bug and left it as outside "smallest change". In my agent I controlled the step and token budgets, which the fixer file could not set. The vendor's agent needed only a short file, not a loop. It ran headless with pre-approved permissions.
