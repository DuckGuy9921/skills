# company-meeting

A Claude Code skill that simulates a short company meeting. Role-based agents (CEO, CTO, Product, Design, Marketing, Finance) each give their view on a topic, and the result is a synthesized set of positions, decisions, and action items.

## Usage

```
/company-meeting <topic>
/company-meeting pricing for the Pro plan
/company-meeting pricing with CEO, Finance
/company-meeting launch timing with Product, Marketing
```

- Without a topic, the skill asks one clarifying question.
- Naming roles after `with` limits attendance to those roles.

## Default roles

- CEO
- CTO
- Product
- Design
- Marketing
- Finance

## How it works

Each attending role is answered by a separate subagent running on Haiku. The subagents run in parallel, each giving a short position (under 150 words). The main session then synthesizes them into a markdown summary with positions, agreements, disagreements, decisions, and action items with owners.

Haiku is used for speed and cost, since each role only needs a focused, short answer.
