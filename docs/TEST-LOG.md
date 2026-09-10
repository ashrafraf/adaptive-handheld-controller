# Test Log

Every test that was actually run, whether it passed or not. This is the evidence that closes a stage — a stage is not done because it feels done, it is done because a row here says so.

Record the test **before** interpreting it. If a test was not run, it has no row.

Result values: **PASS** · **FAIL** · **PARTIAL** · **INCONCLUSIVE**

| ID | Date | Stage | Subsystem | Test | Setup / conditions | Result | Evidence | Run by |
|---|---|---|---|---|---|---|---|---|
| | | | | | | | | |

## How to write a row

- **Test** — what was measured, not what was hoped. "3V3 rail measured at test point TP2 under full input load", not "power works".
- **Setup / conditions** — enough for someone else to repeat it: board revision, firmware commit, supply, instrument, ambient conditions.
- **Result** — the number or the observation, then PASS/FAIL against a stated criterion. A test with no pre-stated pass criterion is an observation, not a test.
- **Evidence** — path to a screenshot, scope capture, log file or photo in `assets/` or `tests/`.

A FAIL row is not deleted or edited when the problem is fixed. Add a new row for the re-test and log the cause and fix in `FAILURES.md`.
