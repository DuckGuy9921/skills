---
name: decision-memo
description: Use when the user invokes /decision-memo or asks for a one-page decision memo or lightweight ADR, such as "write up this decision", "compare these options and recommend one", or "record why we chose X over Y".
---

# Decision Memo

Turn a pending or made decision into a one-page memo covering context, options, criteria, tradeoffs, a recommendation, risks, and reversibility.

## Steps

1. **Pin down the decision.** Take the text after `/decision-memo` as the decision question. If there is no text, ask one clarifying question: "What must be decided, and by when?" Then stop until the user answers. If the user gives options but little context, proceed and record the gaps under Assumptions.

2. **Gather real context.** When the decision touches code, run `git log --oneline -20` and, for each option, `grep -rnF --exclude-dir=.git -- "<option name>" . | head -n 20` (`-F` treats the name as literal text), noting in the memo when the output hit the 20-line cap. Use this to find existing usage, constraints, and versions. If the user pasted a long document, extract only the facts that bear on the criteria. Run `ls docs/adr docs/decisions adr 2>/dev/null` to find an existing decision log and reuse its numbering and template. Never state a codebase fact you have not checked; unverified points go under Assumptions.

3. **Define options.** List two to four realistic options. Include the status quo when it is viable. Give each option one line describing what choosing it means.

4. **Set criteria before scoring.** Write three to five criteria that actually separate the options (for example cost, delivery time, operational load, security, reversibility). Mark each high, medium, or low weight. Fix the criteria before you judge the options, so the recommendation follows from them.

5. **Weigh tradeoffs.** Build one table with options as rows and criteria as columns. Each cell is a short phrase naming the benefit or cost. Do not invent numeric scores unless the user supplied numbers.

6. **Recommend one option.** Pick a single option and justify it in two or three sentences that cite the high-weight criteria. State what new fact would change the recommendation.

7. **Assess risks and reversibility.** List the top three risks, each with a mitigation. Classify reversibility as easy, costly, or one-way, and name the concrete cost of reverting, such as migration work, lock-in, or data loss.

8. **Parallelise only when it helps.** If the decision needs research on several external products and you can spawn subagents, spawn one per option with `model: haiku`, each returning under 150 words, and run them in a single batch. Otherwise, or for small decisions, work inline.

9. **Save only if a convention exists.** If Step 2 found a decision log, write the memo there as the next numbered file, using its template and status values, and do not commit it. If the log already has an entry for this same decision (same question, matching title or options), write the new memo as the next number, change only the old entry's status to Superseded with a link to the new one, and link the new memo back to it. If you cannot tell whether an entry matches, treat it as unlogged and say so under Assumptions. Otherwise return the memo in chat only.

## Output format

```
# <Decision title>

**Status:** Proposed (or Accepted only if the user states the decision is final) | **Date:** <today's date from the environment> | **Deciders:** <roles or names, or "unspecified">

## Context
Two to four sentences on the problem and why it needs deciding now.

## Options and tradeoffs
<table: options as rows, criteria as columns, with weights in the header>

## Recommendation
<chosen option, two or three sentences, and what would change it>

## Risks and mitigations
- <risk>: <mitigation>  (three bullets)

## Reversibility
<easy | costly | one-way>, with the concrete cost of reverting.

## Assumptions and open questions
- <bullet per unverified point or unanswered question>
```

## Notes

- Keep it to one page, about 500 words. If it runs long, drop the lowest-weight criterion first, never the risks.
- Recommend, do not decide. Leave Status as Proposed and name the deciders the user gave, unless the user states the decision is final.
- Superseding follows Step 9. Never delete the old memo.
- Do not send, post, or publish the memo anywhere. Create no files beyond the single memo; the only other permitted change is the Superseded status edit in Step 9. Never run `git commit`, `git push`, or delete anything.
- Do not guess dates, versions, or prices. Use the date from the environment and mark anything else unverified.
