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
- Performance optimization

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
- **inefficient implementation** — correct output, suboptimal approach (e.g., O(n²) search fixable with a hash map, recomputing an aggregation inside a loop, sorting to find min/max instead of a single-pass scan). All correctness tests must pass against the unoptimised code. Use this for `Performance optimization` category challenges.

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
- **Easy**: single file; function body ≥15 lines of logic; bug is a domain logic error (not a typo or obviously wrong constant); generic algorithm names (e.g., `sliding_window`) only allowed if wrapped in a domain scenario with domain-specific variable names
- **Medium**: 2–3 files; each helper ≥10 lines of real computation (not trivial validators); bug can only be pinpointed by understanding how the main function and at least one helper interact; variable names must reflect domain concepts throughout (no `n`, `arr`, `i`, `result` as primary domain variables)
- **Hard**: 3–4 files; no standalone algorithm functions — all logic must be embedded in domain scenarios; bug is in a helper and only understandable via the main file's contract; the defective line must look like correct, competent code; reading the main file alone must be insufficient to identify the bug

**Complexity quality bar — apply these two tests before finalising a challenge design:**
- **The domain test**: "Could you find this bug knowing only the algorithm, without understanding the business domain?" If yes, redesign — the bug must require domain knowledge
- **The unremarkable line test**: "Does the defective line stand out syntactically or structurally?" If yes, make the bug more subtle or move it deeper

**Anti-patterns — never generate:**
- A function named after an algorithm at medium/hard (e.g., `sliding_window`, `binary_search`) — embed the algorithm in a domain scenario
- A bug where the variable name makes the error self-evident (e.g., `maxSize + 1` in a function called `get_window_size`)
- Helper files that are purely input validators — they must contain real computation the main function depends on
- A challenge where the defective line is the most syntactically interesting line in the file
- Generic variable names (`n`, `arr`, `i`, `result`) as the primary domain variables in medium/hard challenges

---

#### Efficiency Challenge Rules (defect type: `inefficient implementation`)

When generating a `Performance optimization` category challenge:

- All correctness tests must pass against the unoptimised code — it is functionally correct
- The task description must explicitly state current and target complexity (e.g., "Currently O(n²) — optimise to O(n)")
- Omit the "at least 1 test that fails against the buggy implementation" requirement — instead, all correctness tests must pass
- Generate a **separate performance test file** alongside the regular test file:
  - Python: `perf_test_<challenge_name>.py` in `python/tests/`
  - TypeScript: `<challengeName>.perf.ts` in `typescript/tests/`
  - Go: `<challenge_name>_bench_test.go` co-located in the challenge directory (uses `testing.B`)
  - Java: `<ChallengeName>PerfTest.java` in `java/src/test/java/challenges/<packageName>/`
- The performance test runs the function with a large representative input — choose N so the inefficient version takes approximately 500ms–2s — and records the median time over 5 runs
- Performance test files must be excluded from the regular test runner (`/check` and `/run-suite` must not execute them — they are only run via `/benchmark`)
  - Python: prefix with `perf_test_` (not `test_`) so pytest ignores them by default
  - TypeScript: use `.perf.ts` extension; add `exclude: ['**/*.perf.ts']` note in vitest config if needed
  - Go: use build tag `//go:build bench` at the top of the bench file
  - Java: annotate with `@Tag("perf")` — excluded from the default Maven Surefire run

---

#### Test File Rules

- At least 1 happy-path test
- At least 2 edge cases
- At least 1 test that will **fail** against the buggy implementation (proving the bug is real) — omit for `inefficient implementation` challenges
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
