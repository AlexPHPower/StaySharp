---
name: restart
description: This skill should be used when the user runs "/restart" or asks to wipe all current challenges and regenerate fresh ones without archiving results or running tests.
version: 1.0.0
---

# Restart

Wipe all active challenges and regenerate a fresh set — without running tests or archiving results.

Use this when you want to start over mid-session (e.g., a challenge feels wrong, you accidentally looked at the answer, or you just want a different set).

## Arguments

Same as `/create-suite`:

- `count` — number of challenges to generate **per language** (default: matches the current suite's count, or 4 if none active)
- `languages` — comma-separated list (default: matches the current active languages, or all four if none active)

Example: `/restart` — regenerates same count and languages as the current suite
Example: `/restart count=2 languages=python` — wipes everything, generates 2 Python challenges

## Instructions

### 1. Detect Current Suite

Check which languages have active challenge files (non-`.gitkeep` files):

- Python: any `.py` files in `python/challenges/`
- TypeScript: any `.ts` files in `typescript/challenges/`
- Go: any `.go` files in `go/challenges/` (excluding `*_test.go`)
- Java: any `.java` files in `java/src/main/java/challenges/`

Record the active languages and how many challenge directories each has (to infer the current count).

### 2. Resolve Arguments

If `languages` was provided as an argument, use that. Otherwise use the detected active languages (or all four if none active).

If `count` was provided as an argument, use that. Otherwise use the detected per-language challenge count (or 4 if none active).

### 3. Delete All Active Challenge Files

Delete all challenge subdirectories and test files for the **active languages** (leave `.gitkeep` files intact):

- Python: delete all subdirectories in `python/challenges/` and all `test_*.py` files in `python/tests/`
- TypeScript: delete all subdirectories in `typescript/challenges/` and all `*.test.ts` files in `typescript/tests/`
- Go: delete all subdirectories in `go/challenges/`
- Java: delete all subdirectories in `java/src/main/java/challenges/` and `java/src/test/java/challenges/`

### 4. Confirm Deletion

Print a brief message before regenerating:

```
Restarting suite — wiped N challenges across [languages].
Generating fresh challenges...
```

### 5. Regenerate

Run `/create-suite` with the resolved count and languages.

---

## Constraints

- **Never run tests** — this command does not evaluate anything
- **Never write a results file** — no archiving of any kind
- Leave `.gitkeep` files in place
- If no challenges are currently active, just run `/create-suite` with the provided or default arguments
