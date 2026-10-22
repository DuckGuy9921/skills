---
name: perf-investigation
description: Use when the user invokes /perf-investigation or asks why a command, test, endpoint, function, query or build is slow, wants a performance baseline, a profile of a bottleneck, or proof that a speedup works with before and after numbers.
---

# Performance Investigation

Find the real bottleneck in a slow piece of code by measuring first, changing one thing at a time, and reporting before and after numbers.

## Steps

1. **Pin down the target.** Use the text after `/perf-investigation` as the target. If there is no text, check `git status` and `git log --oneline -5` for an obvious candidate. If none exists, ask the user one question about what is slow, then stop. State the target and the success metric (wall time, p95 latency or peak memory) before measuring. If running the target would write data, call a live or paid service, or delete files, ask the user to confirm or to supply a safe test input first. This confirmation covers every run in steps 2, 3 and 6.

2. **Record the baseline.** Run the target at least 5 times with identical inputs (at least 20 for p95). Time commands with `time`, or `hyperfine` if `command -v hyperfine` finds it. Time functions, endpoints or queries with a small script or `curl -w '%{time_total}'`. Measure peak memory with `/usr/bin/time -v` if installed. If one run exceeds 60 seconds, ask the user for a smaller representative input. Record the median, the min-max range, the commit from `git rev-parse --short HEAD`, and load from `uptime`. Save raw numbers in the scratchpad directory.

3. **Profile before editing.** Run a text-output profiler, saving output in the scratchpad directory: `python -m cProfile -s tottime <script>`, `node --prof <script>` run from the scratchpad directory then `node --prof-process`, `go test -cpuprofile <scratchpad>/cpu.out` then `go tool pprof -top <scratchpad>/cpu.out`, or `EXPLAIN ANALYZE` for SQL. `EXPLAIN ANALYZE` runs the statement, so use a read-only query, or a write inside a transaction you roll back. Rollback does not undo external side effects, so such a write needs step 1 confirmation. Read the top 10 entries by self and cumulative time from the saved output. Do not guess the bottleneck from the code alone.

4. **Confirm one bottleneck.** Pick the single function, query or call with the largest self-time share of total time. For the first bottleneck, if none exceeds 30 percent, profile at finer granularity or with a more representative input before changing anything. If none exceeds 30 percent after that, report that no single hotspot dominates and stop.

5. **Make one change.** Before each change, copy every file it will touch into the scratchpad directory so it can be restored exactly. Edit only the code for that bottleneck with `Edit`. Run the existing tests covering the touched code, such as `pytest <path>` or `npm test`. If they fail, fix the regression or revert before continuing.

6. **Re-measure.** Repeat step 2 with the same inputs and run count, and compare medians. Keep the change only if the new median beats the baseline median by more than the baseline min-max range. Otherwise copy the saved files from step 5 back over the originals. Never use `git checkout --` or `git restore`, which would also discard the user's own uncommitted changes.

7. **Iterate or stop.** The kept median becomes the baseline. Return to step 3 for the next top self-time item. Stop when the success metric is met, or when the newest profile shows no item above 10 percent of the total.

## Output format

```
## Perf investigation: <target>
Metric: <metric>, <N> runs per measurement, median (min-max)

| Iteration | Change | Before | After | Delta |
|-----------|--------|--------|-------|-------|
| 0 | baseline | <median (min-max)> | - | - |
| 1 | <one-line change> | <median (min-max)> | <median (min-max)> | <percent> |

Bottleneck: <function, query or call, percent of total time>
Kept: <changes kept> | Reverted: <changes reverted>
Files changed: <absolute paths>
Remaining hotspots: <top item and percent, or none>
```

## Notes

- Report only measured numbers. Never claim a speedup from a single run or from reading code.
- Do not commit or push unless the user asks.
- If the target needs production data, a remote service or a missing tool, report what is missing rather than inventing numbers.
- If subagents are available, use them with `model: haiku` only for separate, independent targets, under step 1's safety rules. Timing runs in steps 2 and 6 never overlap other work.
