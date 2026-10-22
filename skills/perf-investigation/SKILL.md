---
name: perf-investigation
description: Use when the user invokes /perf-investigation or asks why a command, test, endpoint, function, query or build is slow, wants a performance baseline, a profile of a bottleneck, or proof that a speedup works with before and after numbers.
---

# Performance Investigation

Find the real bottleneck in a slow piece of code by measuring first, changing one thing at a time, and reporting before and after numbers.

## Steps

1. **Pin down the target.** Take the text after `/perf-investigation` as the slow thing: a command, test, endpoint, function or file. If there is no text, run `git status` and `git log --oneline -5` to look for an obvious candidate. If none exists, ask the user one question about what is slow and stop until they answer. State the target and the success metric (wall time, p95 latency or peak memory) before measuring.

2. **Record the baseline.** Run the target at least 5 times with identical inputs, using `time` for commands or `hyperfine` if `command -v hyperfine` succeeds. Record the median and the min-max range. Record the commit with `git rev-parse --short HEAD` and check machine load with `uptime`. Write raw numbers to a scratch file outside the repo, such as under the scratchpad directory.

3. **Profile before editing.** Choose the profiler that matches the language and run it on the target: `python -m cProfile -s cumtime`, `node --cpu-prof`, `go test -cpuprofile`, `perf record` for native binaries, or `EXPLAIN ANALYZE` for SQL. Read the top 10 entries by self time and by cumulative time with `Read` or `grep`. Do not guess the bottleneck from the code alone.

4. **Confirm one bottleneck.** Pick the single function, query or call that accounts for the largest share of the total time. If no single item exceeds about 30 percent, profile at a finer granularity or with a more representative input before changing anything.

5. **Make one change.** Edit only the code for that bottleneck with `Edit`. Make no unrelated cleanups. Run the existing tests covering the touched code, such as `pytest <path>` or `npm test`. If they fail, fix the regression or revert before continuing.

6. **Re-measure.** Repeat step 2 with the same inputs, environment and run count. Compare medians. Keep the change only if the improvement is larger than the baseline min-max range. Otherwise revert the edit with `git checkout -- <file>`.

7. **Iterate or stop.** Return to step 3 for the next bottleneck, still one change at a time. Stop when the success metric is met, or when the newest profile shows no item above 10 percent of the total.

## Output format

```
## Perf investigation: <target>
Metric: <metric>, <N> runs per measurement, median (min-max)

| Step | Change | Before | After | Delta |
|------|--------|--------|-------|-------|
| 0 | baseline | <median (min-max)> | - | - |
| 1 | <one-line change> | <median (min-max)> | <median (min-max)> | <percent> |

Bottleneck: <function, query or call, percent of total time>
Kept: <changes kept> | Reverted: <changes reverted>
Files changed: <absolute paths>
Remaining hotspots: <top item and percent, or none>
```

## Notes

- Report only measured numbers. Never claim a speedup from a single run or from reading code.
- Keep inputs, test data, environment variables and run count identical between the before and after measurements.
- Do not commit or push unless the user asks.
- If the target needs production data, a remote service or a load-testing tool that is not installed, report what is missing instead of inventing numbers.
- Use subagents with `model: haiku` only to profile separate, independent targets in parallel. A single slow path runs sequentially, because each change depends on the last measurement.
