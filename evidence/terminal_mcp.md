# Terminal agent calling the a4-tools MCP server

How this was produced: Claude Code 2.1.318 run headless (`claude -p`) by another Claude Code session, at my request, on 2026-11-03. The text below is generated from its raw stream-json log, so the layout differs from a terminal; the calls, results and replies are unchanged. No person answered prompts: `--allowedTools` pre-approved only `mcp__a4-tools__run_tests` and `mcp__a4-tools__read_file`. Claude Code's internal ToolSearch lookup, which returned no text, is cut.

Prompt:

> Use the a4-tools MCP server: call its run_tests tool, then its read_file tool on discounts.py, and tell me why the failing test fails. Do not change any files.

[session start] model: claude-opus-5-5; MCP servers: [{"name": "a4-tools", "status": "connected", "source": "dynamic"}]

Tool call: mcp__a4-tools__run_tests
```json
{}
```

Tool result:
```
FAILED: 1 of 4 tests did not pass.
ERROR test_lowercase_code_works: ValueError: unknown code: save10
```

Tool call: mcp__a4-tools__read_file
```json
{
  "path": "discounts.py",
  "limit": 200
}
```

Tool result:
```
discounts.py, lines 1 to 20 of 20
   1: """Discount codes for the shop's checkout."""
   2: 
   3: from datetime import date
   4: 
   5: # code: (percent off, last day the code can be used)
   6: CODES = {
   7:     "SAVE10": (10, date(2026, 12, 31)),
   8:     "WELCOME5": (5, date(2026, 11, 30)),
   9: }
  10: 
  11: 
  12: def apply_discount(total_cents: int, code: str, today: date) -> int:
  13:     """Return the order total, in cents, after the discount code is applied."""
  14:     if code == "":
  15:         return total_cents
  16:     if code not in CODES:
  17:         raise ValueError(f"unknown code: {code}")
  18:     percent, last_day = CODES[code]
  19:     discount = total_cents * percent // 100
  20:     return total_cents - discount
```

`test_lowercase_code_works` fails because `apply_discount` matches discount codes case-sensitively. It's the only failure, 1 of 4 tests. No files were changed.

```
ERROR test_lowercase_code_works: ValueError: unknown code: save10
```

- **Cause:** the keys in `CODES` are all uppercase (`"SAVE10"`, `"WELCOME5"`), at `discounts.py:6-9`. The check at `discounts.py:16`, `if code not in CODES`, compares the input string exactly as given.
- **What happens:** `"save10"` doesn't match `"SAVE10"`, so line 17 raises `ValueError`. The function never reaches the discount calculation.
- **Why it's an `ERROR`, not a `FAIL`:** the test hit an uncaught exception rather than a failed assertion.

**Likely fix:** normalize the code before looking it up, for example `code = code.upper()` before line 16, and use that value for the lookup on line 18. I only read `discounts.py`, not the test file, so the test's exact expectation is inferred from its name and the error message.

**Separate bug:** `last_day` is unpacked on line 18 but never used, and `today` is never read either. Expired codes are therefore still accepted. This isn't causing the failure, and the other three tests pass with it in place.
