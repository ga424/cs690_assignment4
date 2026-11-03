# Prompt sent to Claude (claude.ai agent session, model claude-opus-5-5), 2026-11-03

Verbatim, as typed. This one session produced ledger entries 2 to 7; each entry's
`prompts` field points at the part-specific file, which quotes the specification the
assistant was held to.

---

Help me build my Own Coding Agent
"Objective:
For this individual assignment, you will build your own coding agent in Python on the real tool-calling API and use it to fix a bug in a small project. You will write its tools, approval gates and stop rules, the ReAct loop, a judge with Reflexion, and an MCP server. Then you will plug your tools into Claude Code or Antigravity CLI and compare a vendor's agent with your own on the same bug.
Why this Assignment matters in Industry:
Every agent product, from Claude Code to Copilot's agent mode, is a model, tools and a loop. Knowing the loop lets you debug an agent instead of guessing.
Approval gates, budgets and stop rules are how teams keep agents from looping, overspending or touching what they shouldn't.
MCP is the standard way to give agents and IDEs access to tools. The server you write works in Claude Code unchanged.
A judge the agent cannot edit, a trace of every step, and a provenance ledger are what let a reviewer trust an agent's work.
Assignment Instructions:
Clone the assignment repository from GitHub:[git@github.com:md685-njit/Assignment-4.git]
Read HANDOUT.md and README.md before you start. The handout contains every step in detail.
Create a new repository on your GitHub account and push your work there.
Set up your environment by following the README, and run python -m agent check-setup to confirm it passes.
Complete Parts 1 to 8 in the handout. Write the agent loop yourself; do not use an agent framework.
Record runA and runB exactly once each, and check both with check-trace and replay.
Fill in REPORT.md from your own traces, and keep LEDGER.md as you work."
 Please make sure that you use my aise building pipeline and ensure that all requirements are met for the assignment on the following repository on the target repository https://github.com/ga424/cs690_assignment4

Follow-up, sent mid-session:

Please make sure that you are committing changes by features developed showing good sequence of changes.
