# Expert Finder - Context vs Skills (Side by Side)

This file shows the difference between:
- Context markdown: descriptive project documentation.
- Skills markdown: executable operating guidance for an agent.

Source used for conversion:
- Expert Finder - Feature Launch Summary

---

## 1) Context Markdown (What You Have Been Building)

This style explains the product clearly, but it does not strictly control agent runtime behavior.

### Example (context style)

```markdown
# Expert Finder Agent - Feature Launch Summary

## What It Is
An AI-powered agent that finds internal Microsoft experts using live M365 signals.

## Features
- Multi-signal expert search
- Fast search + deep search modes
- Confidence scoring
- Recency weighting
- Org chart validation
- Outreach draft generation
- Source links and topic tags

## Deployment Options
- Copilot CLI: Live
- M365 Declarative Agent: Live
- MCP Server: Planned
```

Why this is context:
- Great for stakeholders, project status, and docs.
- Weak for deterministic execution because triggers, workflow steps, and output contracts are not formalized.

---

## 2) Skills Markdown (How To Make It Operational)

This style turns the same idea into an instruction system the agent can execute consistently.

### Example (skills style)

```markdown
# Skill: Internal Expert Finder

## Intent
Find and rank internal experts for a user topic using M365 signals.

## Inputs
- user_topic (required)
- depth_mode (optional): fast | deep
- time_window (optional): default 6 months

## Triggers
- "find expert"
- "who owns"
- "who can help with"
- "deep search"

## Required Data Sources
1. People profiles
2. Email threads
3. Meetings/calendar
4. Documents/wiki/specs

## Workflow
1. Normalize user topic into 3 to 5 search terms.
2. Run fast mode by default (single cross-signal query).
3. If user asks for deep search, run 3 parallel queries:
   - Profiles query
   - Email query
   - Meetings and docs query
4. Merge candidates and deduplicate by identity.
5. Score candidates:
   - Signal count
   - Recency
   - Role alignment
6. Assign confidence tier:
   - High: 3+ signals and active in last 3 months
   - Medium: 2 signals and active in last 6 months
   - Low: 1 signal or stale evidence
7. Run org validation only when stale or conflicting evidence appears.
8. Generate outreach drafts for top 2 candidates.

## Output Contract
Return in this order:
1. Summary table with Name, Role, Recency, Confidence, Tags.
2. Detailed evidence per candidate with source links.
3. Outreach drafts for top 2.
4. Offer deep search only if current mode is fast.

## Guardrails
- Do not invent people or ownership.
- Mark stale candidates explicitly.
- State uncertainty when evidence is weak.
- Use only permitted tenant data.

## Failure Handling
- If fewer than 2 credible matches are found, ask one clarifying question and rerun.
- If sources conflict, keep both facts and lower confidence.

## Success Criteria
- At least 3 ranked candidates when available.
- Every candidate includes evidence links.
- Top 2 include outreach drafts.
```

Why this is a skill:
- Defines exact triggers, steps, scoring, and output contract.
- Produces repeatable behavior across sessions and platforms.

---

## 3) Quick Diff Summary

| Dimension | Context Markdown | Skills Markdown |
|---|---|---|
| Primary job | Explain | Execute |
| Runtime control | Low | High |
| Structure | Flexible narrative | Fixed sections and rules |
| Reusability | Project-specific | Portable and reusable |
| Testability | Hard | Easy (check triggers, workflow, output contract) |

---

## 4) Practical Rule for Your Projects

- Keep launch docs, reports, and meeting summaries as context markdown.
- Move stable behavior (decision logic, workflow, output format, guardrails) into skills markdown.
- In your skill, explicitly list which context docs to read before execution.
