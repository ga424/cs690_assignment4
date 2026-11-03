# 04 Test

- Artifact ID: TEST-0001 · Status: Complete (tests given by instructor) · Owner: Gus (ga424)

The executable tests exist before implementation and are protected: `tests/` (public,
scripted model, no key) and `judge/acceptance_test.py` (hidden from the agent). Neither
was edited (EV-0001); the grader restores both anyway.

| Test file | Requirements | Rubric |
| --- | --- | --- |
| tests/test_tools.py (16) | REQ-001..006 | A1 |
| tests/test_policy.py (7) | REQ-007..009 | A2 |
| tests/test_loop.py (9) | REQ-010, REQ-011 | A3 |
| tests/test_reflexion.py (5) | REQ-012..014 | A4 |
| tests/test_server.py (3) + `check-mcp` | REQ-015 | A5, B5 |
| judge/acceptance_test.py (6) | fixture spec rules 1-5 | runA/runB verdicts |
| `check-trace`, `replay` | REQ-011, REQ-019 | B1, B2 |

Spec coverage gap (by design of the assignment): the visible fixture tests do not check
rule 4 (expiry) or the mixed-case `Welcome5` case; only the judge does.
