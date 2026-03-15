---
name: run-suite
description: This skill should be used when the user runs "/run-suite" or asks to finish the current session, archive results, and regenerate fresh challenges.
version: 1.0.0
---

# Run Suite

Run all language test suites, archive results, then regenerate fresh challenges.

## Instructions

### 1. Detect Active Languages

Check which languages have actual challenge files (i.e., non-`.gitkeep` files):

- Python: any `.py` files in `python/challenges/`
- TypeScript: any `.ts` files in `typescript/challenges/`
- Go: any `.go` files in `go/challenges/` (excluding `*_test.go`)
- Java: any `.java` files in `java/src/main/java/challenges/`

Record which languages are active and how many challenge files each has.

### 2. Run Test Suites

Run each active language's test suite. **Do NOT abort if one fails** — capture results for all languages.

Run these exact commands (capture full stdout + stderr + exit code for each):

```
# Python
cd python && python -m pytest tests/ -v --tb=short 2>&1

# TypeScript
cd typescript && npm test 2>&1

# Go
cd go && go test ./challenges/... -v 2>&1

# Java
cd java && mvn test -q 2>&1
```

### 3. Parse Results

From each language's output, extract:
- Total tests run
- Tests passed
- Tests failed
- Names of any failing tests

### 4. Record Challenge Metadata

Before deleting files, collect for each challenge directory:
- Challenge name (directory name)
- All files it contains
- Category (from the main entry-point's module-level docstring)
- Difficulty (from the main entry-point's module-level docstring)

### 5. Write Results File

Create `results/YYYY-MM-DD_HH-MM-SS.md` using the current timestamp.

File format:

```markdown
# StaySharp Results — YYYY-MM-DD HH:MM:SS

## Summary

| Language   | Status | Passed | Failed | Total |
|------------|--------|--------|--------|-------|
| Python     | ✅     | 4      | 0      | 4     |
| TypeScript | ❌     | 2      | 1      | 3     |

## Python

### Challenges

| Challenge              | Files                                     | Category               | Difficulty |
|------------------------|-------------------------------------------|------------------------|------------|
| discount_calculator    | discount_calculator.py, tax_calculator.py | Financial calculations | medium     |

### Test Output

\`\`\`
<full pytest output here>
\`\`\`

### Failed Tests

None.

---

## TypeScript

### Challenges

| Challenge       | Files                            | Category        | Difficulty |
|-----------------|----------------------------------|-----------------|------------|
| parseUserInput  | parseUserInput.ts, validators.ts | Data validation | easy       |

### Test Output

\`\`\`
<full npm test output here>
\`\`\`

### Failed Tests

- `parseUserInput > returns false for empty string`
```

### 6. Print Terminal Summary

Print a concise summary to the terminal:

```
StaySharp Run — 2024-01-15 14:32:00
────────────────────────────────────
Python      ✅  4/4 passed
TypeScript  ❌  2/3 passed  (1 failed)
────────────────────────────────────
Results saved to: results/2024-01-15_14-32-00.md
```

### 7. Regenerate Challenges

After writing the results file:

1. Delete all challenge subdirectories and test files for the active languages (leave `.gitkeep` files intact):
   - Python: delete all subdirectories in `python/challenges/` and all `test_*.py` files in `python/tests/`
   - TypeScript: delete all subdirectories in `typescript/challenges/` and all `*.test.ts` files in `typescript/tests/`
   - Go: delete all subdirectories in `go/challenges/`
   - Java: delete all subdirectories in `java/src/main/java/challenges/` and `java/src/test/java/challenges/`

2. Run `/create-suite` with the same count and languages as the suite that was just run.

---

## Constraints

- Never abort early — always run all active languages even if one fails
- Always write the results file before deleting challenge files
- Leave `.gitkeep` files in place
- Regeneration must use the same challenge count as the run that just completed
