---
name: postmortem
description: Use when the user invokes /postmortem or asks for a blameless incident postmortem, outage review, or incident writeup with timeline, impact, root cause, 5 whys, and action items with owners.
---

# Postmortem

Write a blameless incident postmortem from the evidence in the repo and the user's account, marking every fact as verified or unknown.

## Steps

1. **Get the incident.** Use the text after `/postmortem` as the incident description. If it is empty and the user has not named an incident file, ask one clarifying question (what broke and when) and stop until they answer.

2. **Collect evidence.** Run `git log --since="<window start>" --until="<window end>" --oneline` using the window the user gave to find suspect changes (if no window is given, use `git log -n 50 --oneline`). If `gh` is available, run `gh pr list --state merged --search "<service name>"` for related PRs. Run `grep -rn -m 20` for the exact error text or service name across logs, runbooks, and `incident*.md` files, and cap any long output with `head`. Read any file the user names.

3. **Build the timeline.** List events in UTC: trigger change, first symptom, alert, acknowledgement, mitigation, resolution. Take each timestamp from a commit, log line, or user statement, and record that source. Write `unknown` for any gap.

4. **Quantify impact.** State duration, affected users or services, failed requests, data loss (yes, no, or unknown), and customer-visible symptoms. Use numbers from logs or the user. Never estimate silently; label any estimate as one.

5. **Find the root cause with 5 whys.** Start from the symptom and ask "why" five times, backing each answer with evidence from step 2. Stop early only when you reach a system condition rather than a person's action. The root cause is the condition whose removal would have prevented the incident.

6. **List contributing factors.** Name conditions such as missing tests, unclear alerts, no rollback path, unreviewed config, or slow detection. Describe the system condition, not a person.

7. **Assess the response.** Split observations into what went well (fast detection, clean rollback) and what went poorly (late page, unclear ownership). Give each one evidence line.

8. **Write action items.** For each contributing factor, write one action with a concrete deliverable, a priority (P0, P1, or P2), and an owner. Find the owner in CODEOWNERS or `git blame` if possible, otherwise ask the user. If no owner is known, write `TBD`. Add a due date only if the user gave one.

9. **Check for blame.** Reread the draft. Replace named individuals with roles or systems, and remove any sentence that assigns fault to a person.

## Output format

Reply with markdown in this exact structure:

```
# Postmortem: <incident title>
**Date:** <YYYY-MM-DD> · **Duration:** <value or unknown> · **Severity:** <level or unknown>

## Summary
<two or three sentences>

## Impact
- <bullet per impact dimension>

## Timeline
| Time (UTC) | Event | Source |
|---|---|---|

## Root cause
<one paragraph>

**Why chain:**
1. <why 1>
2. <why 2>
3. <why 3>
4. <why 4>
5. <why 5>

## Contributing factors
- <factor>

## What went well
- <item, with evidence>

## What went poorly
- <item, with evidence>

## Action items
| Action | Priority | Owner | Status |
|---|---|---|---|
```

## Notes

- Never invent timestamps, metrics, names, or commit hashes. Use `unknown` for anything the evidence does not show.
- Do not treat "human error" as a root cause. Ask what system condition allowed the error.
- If evidence is thin, say so in the Summary and limit the root cause to what is verified.
- Return the postmortem in the reply. Do not write it to a file unless the user asks.
- Do not edit code, change config, or deploy while writing the postmortem.
