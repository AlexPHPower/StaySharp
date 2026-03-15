# StaySharp

A regenerative developer kata tool. Claude generates coding challenges (functions with intentional bugs), you fix them, run tests to verify, then archive results and get fresh ones.

---

## Session Start Checklist

**At the start of every session, Claude should:**

1. Check whether language dependencies are installed:
   - Python: `python -m pytest --version` — if it fails, pytest is missing
   - TypeScript: check if `typescript/node_modules/` exists
   - Go: check if `go` is available (`go version`)
   - Java: check if `mvn` is available (`mvn --version`)
2. If any dependencies are missing, proactively say so and offer to run `/setup`
3. Check whether challenge files exist (non-`.gitkeep` files in challenge directories). If challenges are present, remind the user they have an active suite and suggest running tests or `/run-suite`
4. If no challenges exist, suggest running `/create-suite`

---

## What This Repo Is

StaySharp keeps your coding reflexes sharp through deliberate practice. Each session:

1. `/create-suite` — Claude generates challenge directories (buggy functions + supporting files) and paired test files
2. You read the challenge, investigate across the files, find the bug, fix it
3. Run tests manually to verify your fix
4. `/run-suite` — archives your results and generates fresh challenges

No challenges ship in the repo. Everything is generated at runtime.

---

## Directory Structure

```
StaySharp/
├── CLAUDE.md                                    ← you are here
├── .claude/
│   └── skills/
│       ├── setup/SKILL.md                       ← /setup
│       ├── create-suite/SKILL.md                ← /create-suite
│       ├── check/SKILL.md                       ← /check
│       ├── run-suite/SKILL.md                   ← /run-suite
│       ├── restart/SKILL.md                     ← /restart
│       └── reset/SKILL.md                       ← /reset
├── results/                                     ← archived run results (markdown)
│   └── .gitkeep
├── python/
│   ├── pyproject.toml                           ← pytest config
│   ├── challenges/
│   │   └── discount_calculator/                 ← one subdirectory per challenge
│   │       ├── __init__.py
│   │       ├── discount_calculator.py           ← main entry-point
│   │       └── tax_calculator.py                ← supporting file
│   └── tests/
│       └── test_discount_calculator.py          ← one test file per challenge
├── typescript/
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   ├── challenges/
│   │   └── discountCalculator/                  ← camelCase subdirectory
│   │       ├── index.ts                         ← re-exports main function
│   │       ├── discountCalculator.ts            ← main logic
│   │       └── validators.ts                    ← supporting file
│   └── tests/
│       └── discountCalculator.test.ts
├── go/
│   ├── go.mod
│   └── challenges/
│       └── discount_calculator/                 ← its own Go package
│           ├── discount_calculator.go
│           ├── tax_utils.go                     ← supporting file
│           └── discount_calculator_test.go      ← tests co-located (Go convention)
└── java/
    ├── pom.xml
    └── src/
        ├── main/java/challenges/
        │   └── discountCalculator/              ← camelCase subpackage
        │       ├── DiscountCalculator.java
        │       └── TaxCalculator.java           ← supporting class
        └── test/java/challenges/
            └── discountCalculator/
                └── DiscountCalculatorTest.java
```

---

## Slash Commands

### `/setup`

Install all language dependencies from the repo root. Safe to run multiple times.

```
/setup
```

Checks each language's current state, runs installs only where needed, and prints a status summary. Run this once when you first clone the repo.

---

### `/create-suite`

Generates challenge directories with paired test files.

```
/create-suite                              # 4 challenges per language (16 total)
/create-suite count=2                      # 2 challenges per language
/create-suite languages=python,go          # 4 challenges each in Python and Go (8 total)
/create-suite count=2 languages=typescript # 2 TypeScript challenges
```

**Arguments:**
- `count` — number of challenges **per language** (default: 4)
- `languages` — comma-separated: `python`, `typescript`, `go`, `java` (default: all four)

Each run uses varied categories and difficulties (default distribution: 1 easy, 2 medium, 1 hard per language).

---

### `/restart`

Wipes all active challenges and regenerates a fresh set — without running tests or archiving results.

```
/restart                              # same count and languages as current suite
/restart count=2                      # 2 challenges per language, same languages
/restart languages=python,go          # same count, Python and Go only
```

Use this when a challenge feels wrong, you accidentally saw the answer, or you just want a different set.

**Arguments:**
- `count` — per language (default: matches current suite, or 4)
- `languages` — comma-separated (default: matches current active languages, or all four)

---

### `/check`

Runs all active test suites and reports pass/fail per challenge. Nothing is archived, deleted, or regenerated — use this for a mid-session progress check.

```
/check
```

---

### `/run-suite`

Runs all test suites, archives results, regenerates fresh challenges.

```
/run-suite
```

No arguments. Detects which languages have active challenge files and runs only those.

**What it does:**
1. Detects active languages (languages with non-`.gitkeep` files)
2. Runs each language's test suite (never aborts on failure)
3. Parses pass/fail counts
4. Writes a results file to `results/YYYY-MM-DD_HH-MM-SS.md`
5. Prints a terminal summary
6. Deletes challenge directories and test files, then regenerates with the same count/languages

---

## Setup

Run `/setup` or manually:

```bash
# Python
pip install pytest

# TypeScript
cd typescript && npm install

# Go (no extra deps — standard library only)
cd go && go mod tidy

# Java
cd java && mvn install -DskipTests
```

---

## Running Tests Manually

```bash
# Python
cd python && python -m pytest tests/ -v --tb=short

# TypeScript
cd typescript && npm test

# Go
cd go && go test ./challenges/... -v

# Java
cd java && mvn test -q
```

---

## Challenge Structure

### Multi-File Challenges

Each challenge lives in its own subdirectory and may span 1–4 files. The intent is to mirror a real codebase: the main function delegates to helpers, validators, formatters, or data models. The developer must trace logic across files to find the bug.

- **Main/entry file**: contains the public function the tests call; has the module-level docstring with challenge metadata (name, category, difficulty, defect type, task description)
- **Supporting files**: named after what they do (`tax_calculator.py`, `validators.ts`, `date_utils.go`) — these are genuinely correct; they are not decoys
- **The bug**: lives in exactly one file (could be main or a helper); **no comment or marker identifies it** — the defect type appears only in the main file's docstring

### Difficulty and File Count

| Difficulty | Typical file count | Bug location                          | Complexity bar                                      |
|------------|--------------------|---------------------------------------|-----------------------------------------------------|
| easy       | 1                  | Main file                             | Requires understanding the function's domain logic  |
| medium     | 2–3                | Main or one helper                    | Must read ≥2 files to understand full flow          |
| hard       | 3–4                | Buried in a helper (required)         | Domain-logic-level bug; delegation chain must be traced |

---

## Challenge Categories

Challenges are drawn from these categories (max one per category per language per run):

| Category                  | Example                                         |
|---------------------------|-------------------------------------------------|
| Algorithm                 | Binary search, sorting, graph traversal         |
| String manipulation       | Parsing, formatting, pattern matching           |
| Business logic            | Discount tiers, eligibility rules, workflows    |
| Financial calculations    | Interest, tax, currency rounding                |
| Data validation           | Input sanitization, format checks               |
| Date/time business logic  | Business days, deadlines, scheduling            |
| Data transformation       | Aggregation, filtering, reshaping               |

---

## Defect Types

Each challenge has exactly one intentional defect (across all its files):

| Defect type              | Example                                    |
|--------------------------|--------------------------------------------|
| Logic bug                | `and` instead of `or`                      |
| Off-by-one               | `<` instead of `<=`                        |
| Wrong formula            | Divides by 100 instead of 1000             |
| Incomplete implementation| Returns early, skips a case                |
| Inverted condition       | `not is_valid` when `is_valid` is correct  |

The defect type is recorded only in the main/entry file's module docstring. No marker, comment, or annotation appears in the source code on or near the defective line.

---

## Results Files

Stored in `results/` as `YYYY-MM-DD_HH-MM-SS.md`. Never deleted — they accumulate as your history.

Each file contains:
- Summary table (language, status, passed, failed, total)
- Per-language section: challenge list (name, files, category, difficulty), full test output, failed test names

---

## Constraints for Claude When Generating

- Never put the solution or correct code in any comment
- **Never add any comment that marks, hints at, or is near the defective line** — no `# BUG:`, no suspicious annotations
- Defect type is recorded only in the main/entry file's module docstring
- Bugs must be plausible — a competent developer could have written them; must require understanding domain logic to identify
- Bugs must be non-trivial — the developer should need to trace across files, not just glance at one line
- Exactly **one bug per challenge** — never two defects, never a bug in a helper when the main file already has one
- Hard challenges must place the bug in a helper/supporting file, not the main file
- Supporting files must be correct — they are real logic, not decoys
- Always generate all files for a challenge simultaneously — never partially
- Test names describe behavior, not the bug (no hints)
- At least 1 happy path + 2 edge cases + 1 test that fails against the buggy implementation
- Never overwrite existing challenge directories
