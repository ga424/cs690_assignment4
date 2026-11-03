# 02 Design

## Metadata

- Artifact ID: DESIGN-0001
- Pipeline ID: AISE-DELIVERY-0001
- Status: In review (awaiting owner approval)
- Owner: Gus (ga424)
- Source artifacts: REQUIREMENTS-0001
- Last updated: 2026-11-03

## Architecture

```
 person ──approve(call, diff)──┐
                               ▼
 provider.next_turn ◀── run_agent (loop.py) ──▶ gate_for / budget_stop / is_stuck (policy.py)
   (one API call            │       │
    per step)               │       └──▶ TraceWriter (trace.py): tool / stop / judge / lesson events
                            ▼
                 toolbox.call(name, args)
             LocalToolbox ──▶ agent/tools.py
             MCPToolbox ──stdio──▶ server.py ──▶ agent/tools.py
                            ▲
 reflexion (reflexion.py) ──┴─ judge(workspace) runs judge/ outside the workspace
                              write_lesson → NOTES.md → build_system_prompt next attempt
```

## Decisions (ADRs)

| ID | Decision | Rationale |
| --- | --- | --- |
| ADR-001 | The loop is a plain `while True` over `provider.next_turn`; no framework | HANDOUT AI rule 1; replay must reproduce the same calls |
| ADR-002 | Gates are enforced in code (`gate_for`), not in the prompt | A prompt is a request; the policy is a lock. The model is never told about the tests/ rule |
| ADR-003 | Denied tools are removed from the offered specs as well as refused at call time | Least privilege; defense in depth |
| ADR-004 | `format_report` emits no timings or tracebacks | `is_stuck` compares strings; output must be deterministic |
| ADR-005 | `server.py` reuses `agent.tools.TOOLS` and their docstrings | One source of truth for descriptions across local and MCP toolboxes |
| ADR-006 | Edits are previewed as diffs from the given `toolbox.preview`; un-previewable edits run unasked so the tool returns its own teaching error | Comment in run_agent step 4; avoids asking a person to approve an edit that cannot apply |
| ADR-007 | Reviewer subagent tools limited to Read, Grep, Glob in its header | Lock, not request (HANDOUT Part 7B) |

## Security / Threat Notes

| Threat | Control |
| --- | --- |
| Agent reads or edits the judge (`../judge/...`) | `safe_path` resolves and refuses paths outside the workspace; judge dir is outside every workspace |
| Agent edits tests to make them pass | `gate_for` denies `edit_file` under `tests/` after `\`→`/` and `normpath` |
| Runaway loop / spend | `budget_stop` before every model call; `is_stuck` after each run_tests |
| Prompt injection via a third-party MCP server | Out of scope for code; human approval of edits is the only runtime control. Recorded in REPORT §6 by the owner |
| API key leak | `.env` in `.gitignore`; never printed or committed |
