---
name: company-meeting
description: Use when the user invokes /company-meeting or wants a simulated company meeting where role-based agents (CEO, CTO, Product, Design, Marketing, Finance) discuss a topic and produce decisions and action items.
---

# Company Meeting

Simulate a short leadership meeting on a topic. Each role gives its view, then the meeting is synthesized into decisions and action items.

## Steps

1. **Get the topic.** Use the text after `/company-meeting` as the topic. If there is no topic, ask the user one clarifying question (what the meeting should decide) and stop until they answer. Do not ask more than one question.

2. **Pick attendees.** Default roles: CEO, CTO, Product, Design, Marketing, Finance. If the user names specific roles (e.g. `/company-meeting pricing with CEO, Finance`), use only those roles. Match role names case-insensitively.

3. **Gather role positions in parallel.** Spawn one subagent per attending role, all in a single batch so they run concurrently. Each subagent must:
   - Use `model: haiku`.
   - Receive the topic and its role name, and answer from that role's perspective only.
   - Keep the answer under 150 words.
   - Cover its position, key concern or opportunity, and a concrete recommendation.

4. **Synthesize the meeting.** Once all subagents return, write the meeting notes in markdown:
   - **Positions:** a short bullet or two per role.
   - **Agreement:** points most roles share.
   - **Disagreement:** real tensions between roles, and who holds each side.
   - **Decisions:** the final decisions the meeting reaches, stated as clear one-line decisions.
   - **Action items:** a table or list with columns `Action`, `Owner (role)`.

5. **Keep it concise.** Prefer short bullets over prose. Do not repeat each role's full answer. Do not add a preamble.

## Notes

- Subagent answers are inputs to the synthesis, not final authority. Resolve conflicts in the synthesis and say where a decision is still open.
- If a subagent fails, note the missing role in the output and continue with the rest.
