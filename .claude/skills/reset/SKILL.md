---
name: reset
description: This skill should be used when the user runs "/reset" or asks to wipe all current challenges and return the repo to a clean state without generating new challenges.
version: 1.0.0
---

# Reset

Wipe all active challenges and leave the repo clean — no tests run, no results archived, no new challenges generated.

Use this when you want a completely clean slate before running `/create-suite` yourself with custom options.

## Instructions

### 1. Detect Active Challenges

Check which languages have active challenge files (non-`.gitkeep` files):

- Python: any `.py` files in `python/challenges/`
- TypeScript: any `.ts` files in `typescript/challenges/`
- Go: any `.go` files in `go/challenges/` (excluding `*_test.go`)
- Java: any `.java` files in `java/src/main/java/challenges/`

If nothing is active, print "No active challenges — repo is already clean." and stop.

### 2. Delete All Active Challenge Files

Delete all challenge subdirectories and test files for active languages (leave `.gitkeep` files intact):

- Python: delete all subdirectories in `python/challenges/` and all `test_*.py` files in `python/tests/`
- TypeScript: delete all subdirectories in `typescript/challenges/` and all `*.test.ts` files in `typescript/tests/`
- Go: delete all subdirectories in `go/challenges/`
- Java: delete all subdirectories in `java/src/main/java/challenges/` and `java/src/test/java/challenges/`

### 3. Confirm

Print a brief confirmation:

```
Reset complete — wiped N challenges across [languages].
Run /create-suite to generate a new set.
```

---

## Constraints

- **Never run tests**
- **Never write a results file**
- **Never generate new challenges**
- Leave `.gitkeep` files in place
