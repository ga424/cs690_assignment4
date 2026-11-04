# Reviewer's review of runs/runB/workspace/discounts.py

How this was produced: Claude Code 2.1.318 run headless (`claude -p`) by another Claude Code session, at my request, on 2026-11-03. The text below is generated from its raw stream-json log, so the layout differs from a terminal; the calls, results and replies are unchanged. The reviewer subagent came from `.claude/agents/reviewer.md`; it used only Read and Glob. Claude Code wrote its own task prompt for the subagent from the prompt below.

Prompt:

> Use the reviewer subagent to review runs/runB/workspace/discounts.py against runs/runB/workspace/spec.md.

The reviewer's report, as returned:

**No deviations: `discounts.py` follows all five rules in `spec.md`, and nothing the spec requires is missing.** I checked this by reading and tracing the code by hand. I did not run it or the tests, and I did not review `tests/test_discounts.py`.

| Rule | Spec clause | Status | Evidence |
|---|---|---|---|
| 1 | Empty code leaves the total unchanged | Followed | Lines 14-15 return `total_cents` before any lookup, so `apply_discount(1999, "", d)` returns 1999, even for a date after every code's last day. |
| 2 | Codes are not case-sensitive | Followed | Line 16 applies `.upper()` before the lookup and the `CODES` keys are uppercase, so `"save10"` and `"Save10"` behave like `"SAVE10"`. |
| 3 | Unknown code raises `ValueError` | Followed | Lines 17-18 raise it, for example for `"BOGUS"`. |
| 4 | Usable through the last day, `ValueError` from the next day | Followed | Line 20 uses a strict `>`. `SAVE10` works on `date(2026, 12, 31)` and raises on `date(2027, 1, 1)`. |
| 5 | Discount is whole cents, rounded down | Followed | Line 22 multiplies before the floor division, so it uses exact integer arithmetic with no float error. The spec example gives `1999 * 10 // 100 = 199`, and line 23 returns 1800. |

**Cases the spec doesn't settle**

None of these are rule violations, and only the first looks worth fixing:

1. **Non-ASCII look-alike codes (line 16):** `"ſave10"` (starting with U+017F, long s) is accepted as `SAVE10` because `.upper()` maps `ſ` to `S`. Rule 3 could be read as requiring `ValueError` here, since rule 2 only describes case differences. The impact is low because it only grants a discount that already exists. If you want it strict, you could reject non-ASCII input with `code.isascii()`.
2. **Negative totals (line 22):** `apply_discount(-1999, "SAVE10", d)` computes a discount of -200 and returns -1799. Floor division rounds toward negative infinity, so the discount's size rounds up. The spec doesn't say whether negative totals are valid, and the code doesn't check for them.
3. **`datetime` passed as `today` (line 20):** a `datetime` raises `TypeError` when compared with a `date`, rather than `ValueError`. The signature declares `date`, so this is outside the contract.
4. **Whitespace:** `" SAVE10"` and `"   "` raise `ValueError` as unknown codes. This fits rules 1 and 3 as written, because only `""` counts as empty.

**What the spec can't confirm**

The spec gives no list of codes, so the `CODES` table (lines 6-9) can't be fully checked against it. Only `SAVE10` at 10 percent is implied, by the rule 2 and rule 5 examples. The `WELCOME5` entry and both expiry dates have no source in the spec.

**Files**
- `/Users/gustavoabreu/njit/cs690/cs690_assignment4/runs/runB/workspace/discounts.py`
- `/Users/gustavoabreu/njit/cs690/cs690_assignment4/runs/runB/workspace/spec.md`
