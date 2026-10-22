---
name: study-plan
description: Use when the user invokes /study-plan or asks for a learning plan, study schedule, or roadmap for a topic, skill, or exam, given a goal, current level, weekly time budget, and optional deadline.
---

# Study Plan

Build a realistic, time-boxed learning plan that turns a topic and goal into milestones, spaced practice, resources, and self-test checkpoints.

## Steps

1. **Parse the request.** Take the text after `/study-plan` as the topic. Extract four inputs if present: goal, current level, weekly time budget, and deadline. If the topic itself is missing, ask one clarifying question and stop. Otherwise fill each missing input with a default: goal is "working competence in the topic", level is "beginner", budget is 5 hours per week, deadline is none. Record every default under Assumptions in the output.

2. **Ground it in local files when relevant.** If the topic refers to this repository or a file the user names, use Glob and Read on that material (for example `README.md`, the main manifest, or the named file) so milestones match what actually exists. Do not invent project details. For a general topic, skip this and state that no local files were used.

3. **Fix the horizon.** Count weeks from today's date to the deadline, or use 8 weeks when there is none. Compute total hours as weeks multiplied by weekly budget. Make the goal measurable, for example "write and explain a working binary search tree" rather than "learn data structures".

4. **Draft 3 to 5 milestones.** Order them from prerequisites to the goal. Give each a concrete deliverable the learner can produce or demonstrate, a week range, and an hour estimate. Scale the estimates so they sum to the total hours within 10 percent. Leave no week without a milestone.

5. **Schedule spaced practice.** For each milestone, plan the first study session, then reviews roughly 1, 3, and 7 days later, so earlier material is revisited before it fades. Keep sessions between 25 and 60 minutes and fit them inside the weekly budget. Show one example week as a day-by-day list.

6. **Choose resources.** Name two to four resources per milestone, preferring official documentation, standard textbooks, and well-known courses. If you are not confident a resource exists and is current, leave it out rather than guessing a title or URL. Do not describe a resource's contents unless you know them.

7. **Write self-test checkpoints.** At the end of each milestone, write a closed-book check: three to five questions, or one task to finish without notes. Set a pass rule, such as completing the task unaided or answering four of five correctly. Set a fail rule: repeat the sessions for the weakest items before moving on.

8. **Check feasibility.** If the budget is under 1 hour per week, or the deadline cannot fit the milestones at the stated level, say so plainly and state what can realistically be covered.

## Output format

```
# Study Plan: <topic>

**Goal:** <measurable goal>
**Assumptions:** <defaults used, or "none">
**Budget:** <hours per week> x <weeks> weeks = <total hours> hours

## Milestones

| Week | Milestone | Deliverable | Hours |
|------|-----------|-------------|-------|

## Example week

<day-by-day list with session lengths and review sessions>

## Resources

<per milestone, 2 to 4 items>

## Self-test checkpoints

<per milestone: questions or task, plus pass and fail rules>

## Feasibility

<one or two sentences, or "On track.">
```

## Notes

- Deliver the plan in chat. Do not write a file unless the user asks for one.
- Do not spawn subagents. This is a single-pass planning task.
- Do not add generic motivation or study tips that are not tied to the topic, and do not use emojis.
- Fit the plan to the stated budget. Do not add extra milestones to look thorough.
- With no stated level, plan from beginner and say so in Assumptions. If the user states a level, start the first milestone there instead of repeating basics.
- Never invent statistics or claims about learning research to justify the schedule. The spacing intervals are a planning choice, so present them as one.
