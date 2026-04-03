# Skill: Internal Expert Finder

> **Version:** 1.0  
> **Data backend:** WorkIQ (`ask_work_iq`)  
> **Platform:** GitHub Copilot CLI with WorkIQ MCP  
> **Sharing:** Drop this file into your workspace and reference it from `.github/copilot-instructions.md`

---

## Setup

Add this line to your `.github/copilot-instructions.md`:

```markdown
Use and follow: path/to/Expert Finder Skill.md
```

Replace the `[PLACEHOLDERS]` in the Customization section below, and you're live.

---

## Customization

| Placeholder | Replace With | Example |
|---|---|---|
| `[TEAM]` | Your team or org name | Platform Engineering |
| `[SENDER_NAME]` | Your name (outreach signature) | Jane Doe |
| `[DOMAIN_TOPICS]` | 3–6 example topics for your domain | Kubernetes, CI/CD pipelines, Terraform |

---

## Intent

Find and rank internal Microsoft experts for a given topic using live M365 signals via WorkIQ.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `user_topic` | Yes | — | The subject, technology, project, or problem to find experts for |
| `depth_mode` | No | `fast` | `fast` (1 query) or `deep` (3 parallel queries) |
| `time_window` | No | 6 months | How far back to search for activity signals |

## Triggers

Activate this skill when the user says any of:

- "find expert on …"
- "who knows about …"
- "who owns …"
- "who can help with …"
- "who should I talk to about …"
- "deep search: …"
- "expert search …"

---

## Required Data Source

**WorkIQ** — call `ask_work_iq` for every query. WorkIQ searches across:

1. **People profiles** — titles, "ask me about" fields, listed skills
2. **Email threads** — senders, recipients, CC'd participants, thread subjects
3. **Calendar / Meetings** — organizers, attendees, recurring series, presentation decks
4. **Documents** — authors, editors, SharePoint/OneDrive files, wiki pages

Do **not** use Microsoft Graph API directly. WorkIQ is the only data source for this skill.

---

## Workflow

### Step 1 — Normalize the Topic

Extract 3–5 search terms from the user's request. If the request is vague (no product area, timeline, or geography), ask **one** concise clarifying question before proceeding.

### Step 2 — Search (Fast Mode, default)

Construct **one** WorkIQ query:

```
Find Microsoft employees who are experts on [topic]. Search across:
(1) People profiles and "ask me about" areas
(2) Recent email threads (last [time_window])
(3) Recent meetings and presentations
(4) Documents, specs, or wikis they authored or own
For each person, return: current role, team, org, which signal types they
appeared in, how recently active, and source links.
Rank by signal count and recency.
```

### Step 3 — Search (Deep Mode, user-triggered)

When the user requests deep search, run **3 parallel** `ask_work_iq` calls:

| Query | Focus | What to ask |
|---|---|---|
| A — Profiles | People + Skills | "Who lists [topic] on their profile, skills, or 'ask me about'? Include role, team, org." |
| B — Email | Collaboration signals | "Who sends, receives, or is CC'd on email threads about [topic] in the last [time_window]? Include thread context and links." |
| C — Meetings + Docs | Content signals | "Who organizes, presents in, or attends meetings about [topic]? Who authored documents or specs about [topic]? Include links." |

After all 3 return, merge candidates and deduplicate by person identity.

### Step 4 — Score Candidates

For each candidate, compute:

**Signal count** (1–4): How many signal types mention them.

| Signals | Tier |
|---|---|
| 3–4 | High |
| 2 | Medium |
| 1 | Low |

**Recency tier:**

| Last Activity | Label |
|---|---|
| 0–3 months | Currently Active |
| 3–6 months | Recently Active |
| 6–12 months | Possibly Stale — verify before contacting |
| 12+ months | Likely Stale — may have changed roles |

**Confidence = Signal count × Recency alignment × Role fit.**

- **HIGH:** 3+ signals, active in last 3 months, role clearly aligns
- **MEDIUM:** 2 signals, active in last 6 months, role plausibly related
- **LOW:** 1 signal, or activity >6 months, or role unclear

### Step 5 — Conditional Org Validation

Run a follow-up `ask_work_iq` query **only if**:

- Any candidate is labeled Possibly Stale or Likely Stale
- A candidate's role seems misaligned with the topic
- Signals conflict (e.g., profile says X but email threads suggest Y)

If all candidates look current and role-aligned, **skip this step**.

If validation is needed, batch all flagged candidates into a single query:

```
Are these people still in roles related to [topic]?
[Name 1] — [current role]
[Name 2] — [current role]
Check their current team, org, and recent activity.
```

### Step 6 — Generate Outreach Drafts

For the **top 2** candidates, generate a ready-to-send email:

```
Subject: [specific, concise subject line]
---
Hi [Name],

[1–2 sentences: who you are and context]
[1–2 sentences: specific question or need]
[1 sentence: why them — cite specific evidence from the search]
[1 sentence: clear ask with low-friction next step]

Thanks,
[SENDER_NAME]
```

**Tone rules:**
- Peer-to-peer for ICs and PMs
- More formal for senior leaders and VPs
- Always reference specific evidence ("I saw you in the [thread/meeting/doc]…")

---

## Output Contract

Return results in **this exact order**:

### 1. Summary Table

```
| # | Name | Role | Recency | Confidence | Tags |
```

Compact — no signal details in the table.

### 2. Detailed Entries (per candidate)

- Name, role, team/org
- Why they match
- Signal breakdown with source links: `[profile] [email] [meeting] [doc]`
- Recency label
- Confidence with score breakdown: `"[3 signals: profile, email, meeting] — Currently Active"`

### 3. Outreach Drafts

Ready-to-send messages for top 2 candidates.

### 4. Deep Search Offer (fast mode only)

End with:

> 💡 Want more accurate results? I can run a **deep search** with parallel signal queries for broader coverage.

Omit this line when already in deep mode.

---

## Guardrails

- **Never invent** people, ownership, or expertise.
- **Never present** weak evidence as confident — state uncertainty explicitly.
- **Always show** which signals contributed to each recommendation.
- **Flag** stale or misaligned candidates clearly rather than burying the warning.
- **Respect** Microsoft internal data boundaries and user permissions.
- **Use only** data accessible through WorkIQ within the user's tenant.
- **Minimize** sensitive detail unless necessary for the recommendation.

## Failure Handling

| Condition | Action |
|---|---|
| Fewer than 2 credible matches found | Ask one clarifying question (narrow scope, broaden topic, adjust time window) and rerun |
| WorkIQ returns no results | Report clearly; suggest alternative search terms or broader topic framing |
| Conflicting signals for a candidate | Keep both facts visible, lower confidence to Medium or Low, note the conflict |
| User topic is ambiguous | Ask one concise question before searching — do not guess |

## Success Criteria

- At least 3 ranked candidates when data is available
- Every candidate includes evidence links from WorkIQ
- Top 2 include ready-to-send outreach drafts
- Confidence and recency labels are present on every candidate
- Deep search offer appears in fast mode

---

## Example Invocations

```
> Find experts on [DOMAIN_TOPICS]

> Who should I contact about [common escalation topic]?

> Who owns [service/project/process]?

> Deep search: [niche technical topic] experts

> Who can help with [cross-team process]?
```
