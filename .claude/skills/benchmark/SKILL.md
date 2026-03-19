---
name: benchmark
description: This skill should be used when the user runs "/benchmark" or asks to run performance tests, measure execution time for efficiency challenges, or compare optimisation results against a previous benchmark.
version: 1.0.0
---

# Benchmark

Run performance tests for efficiency challenges and compare results against your previous best.

## Purpose

`/benchmark` is paired with the `inefficient implementation` defect type. Use it to:

1. **Establish a baseline** — run before optimising to record the unoptimised time
2. **Confirm improvement** — run after optimising to see before/after and % change

Regular correctness tests (`/check`) do not run performance test files. Only `/benchmark` executes them.

---

## Instructions

### 1. Detect Active Efficiency Challenges

Scan all active challenge main/entry files for `Defect type: inefficient implementation` in their module docstrings.

For each language, the main/entry file locations are:
- Python: `python/challenges/<name>/<name>.py`
- TypeScript: `typescript/challenges/<name>/<name>.ts`
- Go: `go/challenges/<name>/<name>.go`
- Java: `java/src/main/java/challenges/<name>/<Name>.java`

If no efficiency challenges are found, print: `No active efficiency challenges found. Run /create-suite to generate new challenges.` and stop.

---

### 2. Run Performance Tests

For each efficiency challenge found, locate and run its performance test file:

- Python: `python/tests/perf_test_<challenge_name>.py` — run with `python python/tests/perf_test_<challenge_name>.py` (not pytest)
- TypeScript: `typescript/tests/<challengeName>.perf.ts` — run with `npx tsx typescript/tests/<challengeName>.perf.ts`
- Go: `go/challenges/<challenge_name>/<challenge_name>_bench_test.go` — run with `cd go && go test ./challenges/<challenge_name>/... -bench=. -benchtime=5x -run=^$ -tags bench`
- Java: `java/src/test/java/challenges/<packageName>/<ChallengeName>PerfTest.java` — run with `cd java && mvn test -Dgroups=perf -q`

Capture the median execution time (ms) reported by each performance test. Never abort on a single failure — run all performance tests and collect results.

---

### 3. Load Existing Benchmarks

Read `benchmarks/current.json` if it exists. This file stores the latest benchmark result per challenge. Structure:

```json
{
  "string_dedup_python": { "median_ms": 1842, "n": 50000, "timestamp": "2026-03-16T10:30:00" },
  "invoice_aggregator_typescript": { "median_ms": 1203, "n": 10000, "timestamp": "2026-03-16T10:30:00" }
}
```

The key format is `<challenge_name>_<language>` (snake_case, lowercase language).

---

### 4. Compare and Update

For each challenge just benchmarked:

- If a previous result exists: compute `change_pct = ((current - previous) / previous) * 100`
- If no previous result: this is the baseline run

**Storage rule**: only overwrite a stored benchmark if the new run is **faster** (lower ms). If the new run is slower, keep the stored value as the best and show the regression in the output without saving it.

Update `benchmarks/current.json` with the new (faster) values and current timestamp.

---

### 5. Print Summary Table

```
Benchmark Results
=================

| Challenge              | Language   | Previous (ms) | Current (ms) | Change      |
|------------------------|------------|---------------|--------------|-------------|
| string_dedup           | Python     | 1842          | 47           | -97.4% ✓   |
| invoice_aggregator     | TypeScript | —             | 1203         | baseline    |
| order_matcher          | Go         | 380           | 420          | +10.5% (not saved) |
```

- Use `—` for previous when it's the first run (baseline)
- Use `✓` when improvement ≥ 10%
- Append `(not saved)` when the new run is slower than the stored best

---

## Storage Format

`benchmarks/current.json` — one entry per challenge, keyed by `<challenge_name>_<language>`:

```json
{
  "<challenge_name>_<language>": {
    "median_ms": 47,
    "n": 50000,
    "timestamp": "2026-03-17T10:30:00"
  }
}
```

Only the latest **best** result per challenge is kept. No history — only the current best.

---

## Integration with Other Commands

- **`/run-suite`**: when archiving results, include benchmark data for any efficiency challenges in the results markdown file. After challenges are deleted, remove their corresponding entries from `benchmarks/current.json`.
- **`/restart`** and **`/reset`**: after deleting challenge files, remove the corresponding entries from `benchmarks/current.json` for any efficiency challenges that were active.
- **`/check`**: never runs performance test files — correctness tests only.

---

## Constraints

- Never run performance test files during `/check` or `/run-suite` test runs
- Never block on a slow benchmark — if a test takes more than 30 seconds, skip it and note it in the summary
- Performance test files use the naming conventions established at generation time — do not rename them
