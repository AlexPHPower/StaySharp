---
name: create-suite
description: This skill should be used when the user runs "/create-suite" or asks to generate coding challenges, create a new suite of kata challenges, or generate buggy functions to fix.
version: 1.0.0
---

# Create Suite

Generate a fresh set of coding challenges with paired test files.

## Arguments

- `count` — number of challenges to generate **per language** (default: 4)
- `languages` — comma-separated list: `python`, `typescript`, `go`, `java` (default: all four)

Example: `/create-suite count=2 languages=python,typescript` — generates 2 challenges per language (4 total)

## Instructions

### 1. Parse Arguments

Extract `count` and `languages` from the arguments. Defaults: count=4, languages=python,typescript,go,java.

Generate `count` challenges for each language. Total challenges = `count × len(languages)`.

### 2. Select Categories

For each challenge, pick from this list. **Maximum one challenge per category per language per run** — vary them:

- Algorithm
- String manipulation
- Business logic
- Financial calculations
- Data validation
- Date/time business logic
- Data transformation

Also assign a difficulty: `easy`, `medium`, or `hard`. For a set of 4 challenges per language, target this distribution: **1 easy, 2 medium, 1 hard**. Scale proportionally for other counts. Never make them all the same difficulty.

### 3. Design the Challenge Structure

Each challenge lives in its **own subdirectory**. A challenge may span **1–4 files** depending on complexity. The goal is to make the developer investigate rather than just glance at one function.

**Decide the structure before writing any code:**

- **Single-file** (simple challenges): one file, one bug, straightforward
- **Multi-file** (medium/hard challenges): a main entry-point file plus 1–3 supporting files with helper functions, validators, formatters, or data models. The bug lives in **exactly one** of the files — main or helper. The others are correct.

Multi-file challenges are more interesting: the developer must read the main function, understand what it delegates to helpers, trace the logic across files, and find where the contract breaks.

---

### 4. Generate Challenge + Test Files

For each challenge, generate **all files simultaneously** (main, helpers, and test) before moving to the next challenge.

---

#### Challenge File Rules

**Exactly one defect exists per challenge** — across all files combined. Pick one defect type per challenge (vary across the run):

- **logic bug** — wrong operator or condition (e.g., `and` instead of `or`)
- **off-by-one** — boundary is wrong by exactly one (e.g., `<` instead of `<=`)
- **wrong formula** — correct-looking calculation with a wrong constant or operation
- **incomplete implementation** — function returns early or skips a case
- **inverted condition** — condition is negated when it shouldn't be

**Main/entry file** must include at the top:
- A module-level docstring with: challenge name, category, difficulty, defect type, and a clear task description. Format:
  ```
  Challenge: discount_calculator
  Category: Financial calculations
  Difficulty: medium
  Defect type: wrong formula
  Task: Implement a discount calculator that applies tiered discounts...
  ```

**The file containing the bug**:
- Contains the defect with **no comment, marker, or annotation** on or near the defective line
- The bug must look like plausible, competent code — not an obvious typo or clearly wrong value

**Supporting files** (helpers, validators, etc.):
- Have clear, correct docstrings describing what each function does
- Are genuinely correct — no bugs, no hints, no markers
- Should feel like real utility modules (not obviously named "helpers")
- Name them after what they do: `tax_calculator.py`, `validators.py`, `date_utils.py`, `formatter.ts`, etc.

**Complexity requirements by difficulty:**
- **Easy**: single file; bug requires understanding the function's logic, not just spotting a typo
- **Medium**: 2–3 files; bug requires reading the main file and at least one helper to understand the full flow; the bug could plausibly be in either
- **Hard**: 3–4 files; bug is buried in a helper file; the developer must trace through the main function's delegation chain to reach it; the bug must be domain-logic-level, not syntactic

---

#### Test File Rules

- At least 1 happy-path test
- At least 2 edge cases
- At least 1 test that will **fail** against the buggy implementation (proving the bug is real)
- Use idiomatic test style for each language (pytest functions for Python, `describe`/`it` or `test` for TS, `t.Run` subtests for Go, `@Test` methods for Java)
- Test the **public interface** of the challenge (import from the main entry-point) — tests should not need to know about supporting files
- Test names must describe **behavior**, not the bug — no hints in test names (e.g., `test_returns_correct_total`, not `test_off_by_one_fixed`)

---

#### Directory Structure and Naming Conventions

Each challenge is a subdirectory. Name subdirectories after the challenge in each language's conventional case.

**Python** — `snake_case` subdirectory:
```
python/challenges/discount_calculator/
    __init__.py                          ← exports the main function
    discount_calculator.py               ← main entry-point (has module docstring)
    tax_calculator.py                    ← supporting file (example)
python/tests/
    test_discount_calculator.py          ← imports from challenges.discount_calculator
```

**TypeScript** — `camelCase` subdirectory:
```
typescript/challenges/discountCalculator/
    index.ts                             ← main entry-point (re-exports main function)
    discountCalculator.ts                ← main logic
    validators.ts                        ← supporting file (example)
typescript/tests/
    discountCalculator.test.ts           ← imports from ../challenges/discountCalculator
```

**Go** — `snake_case` subdirectory (its own package):
```
go/challenges/discount_calculator/
    discount_calculator.go               ← main logic, package discount_calculator
    tax_utils.go                         ← supporting file (example), same package
    discount_calculator_test.go          ← tests, package discount_calculator_test
```

**Java** — `PascalCase` challenge name, `camelCase` subpackage:
```
java/src/main/java/challenges/discountCalculator/
    DiscountCalculator.java              ← main class (has Javadoc with challenge info)
    TaxCalculator.java                   ← supporting class (example)
java/src/test/java/challenges/discountCalculator/
    DiscountCalculatorTest.java          ← tests
```

---

### 5. Output Summary

After generating all files, print a summary table:

```
| # | Language   | Challenge               | Files                                              | Category               | Difficulty | Defect type        |
|---|------------|-------------------------|----------------------------------------------------|------------------------|------------|--------------------|
| 1 | Python     | discount_calculator     | discount_calculator.py, tax_calculator.py          | Financial calculations | medium     | wrong formula      |
| 2 | TypeScript | parseUserInput          | parseUserInput.ts, validators.ts                   | Data validation        | easy       | inverted condition |
| 3 | Go         | date_range              | date_range.go                                      | Date/time logic        | hard       | off-by-one         |
```

---

## Constraints (always follow)

- Never put the solution or the correct code in any comment
- **Never add any comment that marks, hints at, or is near the defective line** — no `# BUG:`, no `# TODO:`, no suspicious inline comments
- Defect type is recorded only in the main/entry file's module docstring — never in source code
- Bugs must be plausible — a competent developer could have written them; they should require understanding domain logic to identify
- Bugs must be subtle — a developer should need to trace logic across files, not just glance at one line
- Exactly **one bug per challenge** — never two defects in the same challenge, never a bug in a supporting file when there's already one in the main file
- Hard challenges must place the bug in a helper/supporting file, not the main file
- Supporting files must be genuinely correct — they are not decoys, they are real (correct) logic
- Always generate all files for a challenge (main, helpers, test) — never partially
- Never overwrite existing challenge files — if a directory already exists, skip it and note it in the summary
