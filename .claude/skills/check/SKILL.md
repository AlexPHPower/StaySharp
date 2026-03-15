---
name: check
description: This skill should be used when the user runs "/check" or asks to check the current status of challenges, run tests without archiving, or see which tests are passing or failing.
version: 1.0.0
---

# Check

Run all active test suites and report results. Does not archive, delete, or regenerate anything.

## Instructions

### 1. Detect Active Languages

Check which languages have actual challenge files (non-`.gitkeep` files):

- Python: any files in subdirectories under `python/challenges/`
- TypeScript: any files in subdirectories under `typescript/challenges/`
- Go: any `.go` files (excluding `*_test.go`) in subdirectories under `go/challenges/`
- Java: any `.java` files in subdirectories under `java/src/main/java/challenges/`

If no active languages are found, tell the user no challenges exist and suggest `/create-suite`.

### 2. Run Test Suites

Run each active language's test suite. **Do NOT abort if one fails** — run all and collect results.

```bash
# Python
cd python && python -m pytest tests/ -v --tb=short 2>&1

# TypeScript
cd typescript && npm test 2>&1

# Go
cd go && go test ./challenges/... -v 2>&1

# Java
cd java && mvn test -q 2>&1
```

### 3. Print Summary

Print a clear summary showing the state of each challenge:

```
StaySharp Check — 09:15:00
──────────────────────────────────────────────
Python
  ✅  discount_calculator   4/4 tests passing
  ❌  date_range_validator  2/3 tests passing
        FAILED: test_excludes_weekends

TypeScript
  ✅  parseUserInput        3/3 tests passing

Go
  ❌  inventory_counter     1/4 tests passing
        FAILED: TestHandlesZeroStock
        FAILED: TestNegativeQuantityReturnsError
        FAILED: TestTotalWithMixedItems

──────────────────────────────────────────────
Overall: 10/14 passing across 4 challenges
When you're ready to finish: /run-suite
```

Map failing test names back to their challenge where possible (by matching test file names to challenge directories).

### 4. Offer Guidance

If any tests are failing, close with:
> Fix the failing challenges, then run `/check` again to verify — or `/run-suite` when you're done.

If all tests pass:
> All challenges solved. Run `/run-suite` to archive your results and get a fresh batch.

---

## Constraints

- Never delete, modify, or archive any files
- Never regenerate challenges
- Output only — read and run, nothing else
