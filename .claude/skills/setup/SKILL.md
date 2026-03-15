---
name: setup
description: This skill should be used when the user runs "/setup" or asks to set up language environments, install dependencies, or prepare the StaySharp development environment.
version: 1.0.0
---

# Setup

Set up all language environments from the repo root. Safe to run multiple times — skips languages that are already configured.

## Instructions

### 1. Check Each Language

Before running any install, check the current state of each language environment:

- **Python**: run `python -m pytest --version 2>/dev/null` — if it exits non-zero, pytest is not installed
- **TypeScript**: check whether `typescript/node_modules/` directory exists
- **Go**: always run `go mod tidy` — it's idempotent and fast
- **Java**: check whether `~/.m2/repository/org/junit/` exists — if not, deps haven't been downloaded

### 2. Run Setup for Each Language

Run these from the **repo root**. Do not abort if one fails — attempt all languages and report results for each.

```bash
# Python
pip install pytest

# TypeScript
cd typescript && npm install && cd ..

# Go
cd go && go mod tidy && cd ..

# Java
cd java && mvn install -DskipTests -q && cd ..
```

### 3. Verify Each Language

After installing, verify each one works by running a quick health check:

```bash
# Python
python -m pytest --version

# TypeScript
cd typescript && npx vitest --version && cd ..

# Go
cd go && go build ./... && cd ..

# Java
cd java && mvn validate -q && cd ..
```

### 4. Print Setup Summary

Print a summary table showing the result for each language:

```
StaySharp Setup
───────────────────────────────────────────
Python      ✅  pytest 8.x.x ready
TypeScript  ✅  vitest 2.x.x ready
Go          ✅  go 1.23 ready
Java        ❌  mvn not found — is Maven installed?
───────────────────────────────────────────
Ready to use: /create-suite count=5 languages=python,typescript,go
```

For any language that failed, include a helpful diagnostic message (missing runtime, wrong version, etc.).

### 5. Offer Next Step

If at least one language is ready, suggest: `Run /create-suite to generate your first challenges.`

---

## Constraints

- Always attempt all languages even if one fails
- Never modify pyproject.toml, package.json, go.mod, or pom.xml during setup
- If a runtime is missing entirely (e.g., Java not installed), report it clearly rather than erroring out
