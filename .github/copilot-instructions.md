# Expert Finder Agent — Copilot Instructions

## Project Overview

**Expert Finder** is an AI-powered agent that helps Microsoft employees find internal experts by searching across Microsoft 365 signals (People profiles, Email threads, Meetings, Documents/SharePoint authorship, Teams messages) and Azure DevOps signals. Built primarily for the Gaming Security & Anti-Cheat team but works for any domain.

**What it does:**
- Searches M365 data to find people with expertise on a given topic
- Cross-references multiple signal types for confidence scoring
- Uses **document authorship** (SharePoint/OneDrive) as a strong expertise signal
- Classifies candidates by recency (Currently Active → Likely Stale)
- Generates ready-to-send outreach emails for top candidates
- Offers fast default search (1 query) and deep search mode (4 parallel queries)

**Primary user:** Salman Khan (PM, Gaming Security & Anti-Cheat)

## Tech Stack

This is a **no-code, instruction-driven agent** — pure configuration and prompt engineering.

- **Platform:** Microsoft 365 Copilot (Declarative Agent) + Copilot CLI with WorkIQ
- **Manifest Schemas:** Teams App v1.19, Declarative Agent v1.6
- **Data Sources:** Microsoft Graph API (People, Mail, Calendar, Files, Teams Messages), WorkIQ, Azure DevOps MCP (Work Items, Pull Requests, Repos)
- **Capabilities (Declarative Agent):** OneDriveAndSharePoint, People (with related content), Email, Meetings, TeamsMessages, WebSearch, GraphConnectors
- **Capabilities (CLI):** WorkIQ (`ask_work_iq`), ADO MCP (work items, PRs, repos, wikis)
- **Deployment:** Teams app package (`appPackage.zip`)

## Architecture

```
Expert Finder/
├── appPackage/                          # Deployment package
│   ├── manifest.json                   # Teams app manifest (v1.19)
│   ├── declarativeAgent.json           # Agent definition + instructions (v1.6)
│   ├── color.png                       # App icon
│   └── outline.png                     # App outline icon
├── expert-finder-instructions.txt      # System prompt (v7, reference copy)
├── skills.md                           # Portable skill template for other teams
├── Expert Finder - Detailed Build Report.md  # Development history & decisions
├── Expert Finder - Feature Launch.md         # Launch summary
├── Expert Finder - Team Meeting Summary.md   # Learnings from building
└── appPackage.zip                      # Built package (ready to deploy)
```

## Key Design Decisions

### Signal Types (7 categories)
- **M365 Signals** (5 types via WorkIQ/Graph):
  1. People profiles and "ask me about" areas
  2. Email threads and collaboration
  3. Meetings and presentations
  4. Documents and SharePoint/OneDrive authorship ★
  5. Teams messages — channel posts, chats, discussion participation
- **DevOps Signals** (1 type via ADO MCP): Work items + Pull requests
- **Meeting Transcript Signals** (1 type via Meeting Transcript Store): Speaker attribution, organizer patterns, action item ownership ★★
- M365 signals = who **talks about** and **creates content** on a topic; DevOps signals = who **builds** it; Transcript signals = who **discusses** it in depth
- ★ Document authorship is a confidence booster: 3+ authored docs = strong expertise signal
- ★★ Transcript speakers/organizers in 2+ meetings = strong domain involvement signal
- ADO signals are a confidence booster: active work item owners and PR authors get elevated ranking

### Declarative Agent Capabilities (v1.6)
The declarative agent uses **granular v1.6 capabilities** instead of the generic `GraphConnectors` for each signal type:

| Capability | Signal Type | Purpose |
|---|---|---|
| `OneDriveAndSharePoint` | Documents (#4) | Search authored docs, specs, wikis in SharePoint/OneDrive |
| `People` | Profiles (#1) | People profiles with `include_related_content: true` for related docs/emails |
| `Email` | Email (#2) | Search email threads for collaboration patterns |
| `Meetings` | Meetings (#3) | Search meeting attendance, presentations |
| `TeamsMessages` | Teams (#5) | Search Teams channel posts and chat threads |
| `WebSearch` | Fallback | Web search for external context |
| `GraphConnectors` | External | External connectors (Jira, ServiceNow, etc.) |

### Search Strategy: Fast Default + Deep Search
- **DEFAULT MODE:** 1 comprehensive M365 query + 1 ADO query + 1 transcript query in parallel (fast, high accuracy)
- **DEEP SEARCH:** 4 parallel M365 queries + 1 ADO query + 1 transcript query (Profiles, Emails, Meetings+Docs, Teams, WorkItems+PRs, Transcript speakers/organizers) — user-triggered
- **Rationale:** Combined queries dilute accuracy (proven in v3 A/B test), parallel queries are slow. Hybrid approach is optimal.

### Confidence Scoring Rubric
- **HIGH (3+ signals):** Appears in 3+ signal types (profile/email/meeting/doc/teams/devops/transcript), active in last 3 months, role clearly aligns. Document authorship (3+ docs), ADO signals, and transcript signals (speaker in 2+ meetings) are strong boosts.
- **MEDIUM (2 signals):** 2 signal types, active in last 6 months, plausibly related role
- **LOW (1 signal):** 1 signal type OR activity >6 months ago

### Recency Tiers
- **Currently Active:** 0-3 months
- **Recently Active:** 3-6 months
- **Possibly Stale:** 6-12 months
- **Likely Stale:** 12+ months

### Conditional Org Chart Validation
Only run follow-up validation if recency is stale OR role seems misaligned. Skip otherwise to save queries.

## Response Format (Always This Order)
1. Summary table: `| # | Name | Role | Org | Org Chain | Recency | Confidence | Tags |`
   - Org Chain: 3 levels up → Person → direct reports. Omit trailing arrow if no reports.
2. Detailed entries with signal breakdowns, document authorship counts, and source links
3. Outreach drafts for top 2 candidates only
4. Deep search offer (default mode only)

## Outreach Email Convention
- Top 2 candidates only (avoid decision fatigue)
- Reference specific evidence ("I noticed you authored several docs on VBS in SharePoint" or "I saw your posts in the anti-cheat Teams channel")
- Peer-to-peer tone for ICs/PMs, more formal for senior leaders
- Clear ask with low-friction next step

## Behavior Rules
- Never invent people, ownership, or expertise
- Never present weak evidence as confident
- Always show which signals contributed to each recommendation
- Flag stale or misaligned candidates explicitly
- Ask clarifying questions when scope is vague (product area, timeline, geography)

## Deployment

### M365 Declarative Agent (Primary)
1. Package `appPackage/` contents → zip (use wildcard `appPackage\*` to avoid nested subfolder)
2. Upload to Teams App Store or Copilot Studio
3. Requires: People.Read, Mail.Read, Calendar.Read, Files.Read, ChannelMessage.Read.All
4. **Note:** v1.6 capabilities are a whitelist. Each capability enables a specific M365 signal. Omit a capability to disable that signal.

### Copilot CLI + WorkIQ + ADO MCP + Meeting Transcript Store
```bash
copilot --mcp workiq
```
Use `ask_work_iq` tool to query M365 data.
ADO MCP tools (`list_work_items`, `repo_list_pull_requests`, etc.) are available when the `azure-devops` MCP server is running. Requires `az login` and Node.js v20+.

Meeting Transcript Store provides speaker-level expertise signals:
```bash
cd "Meeting Transcript Store"
node db.mjs find-experts "[topic]"    # Ranked people with meeting evidence
node db.mjs search "[topic]"          # Full-text search across transcript chunks
```

### MCP Server (Planned)
Not yet implemented. Future phase will expose Expert Finder as a reusable MCP tool.

## Key Domain Topics (Gaming Security)
- SoundWire audio drivers
- ASUS Armoury Crate (anti-cheat compatibility)
- Windows Notification Services (WNS) escalations
- Partner escalations (anti-cheat vendor issues)
- VPN partner onboarding
- Kernel-level driver compatibility

## Development History
7 iterations tested (v1→v7), each solving a specific tradeoff:
- v1: Single query baseline
- v2: Parallel queries (accurate but slow)
- v3: Combined query (fast but inaccurate ❌)
- v4: Parallel + conditional validation (fixed accuracy ✅)
- v5: Fast default + deep search option (best balance ✅)
- v6: M365 Agents Toolkit integration — granular v1.6 capabilities (OneDriveAndSharePoint, People, Email, Meetings, TeamsMessages), document authorship signals, Teams message signals ✅
- v7: Meeting Transcript Store integration — speaker attribution, organizer patterns, action item ownership as 7th signal type (CLI only) ✅
