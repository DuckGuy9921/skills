---
name: sql-helper
description: Use when the user invokes /sql-helper or asks to write, explain, debug, or optimize SQL, including joins, CTEs, NULL handling, indexes, or reading EXPLAIN plans for Postgres, MySQL, SQLite, or another dialect.
---

# SQL Helper

Write, explain, or optimize a SQL query against a confirmed schema and dialect, without running destructive statements unless the user confirms.

## Steps

1. **Confirm the schema and dialect.** Check the request for table definitions and the dialect. If they are missing, run `grep -rniE "create table|create index" --include=*.sql` and check migration folders and ORM models. Infer the dialect from markers such as `SERIAL` or `$1` (Postgres), `AUTO_INCREMENT` (MySQL), or `PRAGMA` (SQLite). If neither settles it, assume ANSI SQL and state the assumption. If the grep finds no schema and the query depends on it, ask one clarifying question and stop.

2. **Classify the task.** Decide whether the user wants a new query (write), an explanation of an existing query (explain), a faster version (optimize), or a fix for wrong results or errors (debug).

3. **Read the existing query first.** For an existing query, locate it with `grep -rn` and open the file with Read. Keep its result set and column names unchanged unless the user asks for a change.

4. **Structure the query with CTEs.** Use one named CTE per logical step instead of nested subqueries. Use explicit `JOIN ... ON` with a join type for every join. Qualify every column with a table alias. Do not use `SELECT *` in a deliverable.

5. **Check NULL pitfalls.** Verify each of these against the query:
   - `col = NULL` and `col <> NULL` are never true; use `IS NULL` or `IS NOT NULL`.
   - `NOT IN (subquery)` returns no rows if the subquery yields any NULL; use `NOT EXISTS`.
   - `COUNT(col)` skips NULLs, while `COUNT(*)` does not.
   - A filter on the right table placed in `WHERE` turns a `LEFT JOIN` into an inner join; move it into the `ON` clause.
   - Wrap nullable values with `COALESCE` only where the default is intended.

6. **Propose indexes.** Look for columns used in `WHERE`, `JOIN ... ON`, and `ORDER BY`. Put equality columns before range columns in a composite index. Check existing indexes in the migrations so you do not duplicate one. State the write-cost trade-off of any new index.

7. **Read the plan.** If a read-only connection is configured, run `EXPLAIN` on a SELECT. Use `EXPLAIN ANALYZE` only for a SELECT, because it executes the statement. For any write statement, do not run EXPLAIN; report the plan as Not run. Flag sequential scans on large tables, large gaps between estimated and actual row counts, nested loops over large inputs, and sorts that spill to disk. Claim a speedup only when the plan supports it.

8. **Guard destructive statements.** Never execute `UPDATE`, `DELETE`, `INSERT`, `DROP`, `TRUNCATE`, or `ALTER` unless the user has explicitly confirmed that exact statement in this conversation. Before asking, show the statement, state the rows it will touch, and give a `SELECT` preview that uses the same `WHERE` clause.

9. **Verify.** If the repo config provides a read-only connection, run the SELECT and check the row count and sample rows against what the user expects, or against a `SELECT COUNT(*)` on the base table when no expectation was given. Otherwise say that the query is unverified.

## Output format

Return exactly these sections, in this order. Write `None` where a section has nothing to report.

```
**Dialect and schema:** <dialect, and the tables/columns used, with the source file or the assumption>
**Query:**
<sql code block>
**Why:** <2-4 bullets on what changed or what the query does>
**NULL and edge cases:** <bullets>
**Indexes:** <proposed DDL with trade-off, or None>
**EXPLAIN findings:** <bullets, or Not run>
**Verification:** <what was run and the result, or Unverified>
**Open items:** <questions or confirmations needed before a destructive step, or None>
```

## Notes

- Never invent table or column names. If the schema is unknown and the query depends on it, ask one clarifying question and stop.
- Never run a destructive statement without explicit confirmation in the same conversation. Running `EXPLAIN` on a write statement is also out of scope.
- Do not print credentials or connection strings from config files in the output.
- If subagents are available and the user asks for a review of many SQL files, handle each file separately and merge the results. Do not spawn subagents for a single query.
