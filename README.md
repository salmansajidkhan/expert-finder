# Expert Finder

**AI-powered internal expert discovery agent that finds subject matter experts across 7 signal types - no code required.**

---

## What Problem It Solves

Finding the right expert inside a large organization is slow and unreliable. People rely on tribal knowledge, manual searches, or asking around - which doesn't scale, fails for new hires, and produces inconsistent results. Expert Finder queries live organizational data across email, meetings, documents, profiles, chat, code repositories, and meeting transcripts to return ranked expert recommendations with confidence scores, evidence links, and ready-to-send outreach messages.

---

## Key Features

- **7 Signal Types** - People profiles, email threads, meetings, document authorship (SharePoint/OneDrive), Teams messages, DevOps activity (work items + PRs), and meeting transcript speaker attribution
- **Dual-Mode Search** - Fast single-query default for speed; user-triggered deep search runs 6 parallel queries across all signal types for maximum coverage
- **Confidence Scoring** - Formalized 3-tier rubric (High / Medium / Low) based on signal count, recency, and role alignment
- **Recency Weighting** - Candidates labeled Currently Active → Recently Active → Possibly Stale → Likely Stale
- **Auto-Generated Outreach** - Top 2 candidates always include a ready-to-send email with evidence-based personalization
- **Source Links** - Every recommendation links back to the original email, document, meeting, or profile for verification
- **Conditional Validation** - Org chart verification runs only when signals suggest stale data, saving unnecessary queries

---

## Architecture

The agent is a **zero-code, instruction-driven system** - pure prompt engineering and platform configuration. No runtime, no backend, no deployment pipeline.

```
User Query
    │
    ▼
┌─────────────────────────┐
│   Expert Finder Agent    │
│   (System Prompt)        │
├─────────────────────────┤
│ Default: 1 M365 query    │
│   + 1 DevOps query       │
│   + 1 transcript query   │
│ Deep: 4 M365 parallel    │
│   + 1 DevOps query       │
│   + 1 transcript query   │
└──────────┬──────────────┘
           │
     ┌─────┼──────┐
     ▼     ▼      ▼
┌────────┐┌──────────┐┌────────────┐
│  M365  ││  DevOps  ││ Transcript │
│Signals ││ Signals  ││   Store    │
│        ││  (MCP)   ││  (local)   │
│ People ││Work Items││ Speakers   │
│ Email  ││Pull Reqs ││ Organizers │
│ Mtgs   ││ Repos    ││ Action Item│
│ Docs ★ ││          ││  Owners    │
│ Teams  ││          ││            │
└───┬────┘└────┬─────┘└─────┬──────┘
    └──────────┼────────────┘
               ▼
┌─────────────────────────┐
│  Rank & Score            │
│  • Signal count (1–7)    │
│  • Recency (4 tiers)     │
│  • Role alignment        │
│  • Confidence (H/M/L)    │
│  • Doc authorship boost  │
│  • DevOps boost          │
│  • Transcript boost      │
└──────────┬──────────────┘
           ▼
┌─────────────────────────┐
│  Output                  │
│  • Summary table         │
│  • Detailed profiles     │
│  • Signal breakdowns     │
│  • Outreach drafts       │
│  • Deep search offer     │
└─────────────────────────┘

★ Document authorship is a strong confidence
  booster - 3+ authored docs = high expertise signal
```

**How it works:** The agent receives a natural language query ("Who knows about Kubernetes security?"), constructs targeted searches against organizational data sources, cross-references results across signal types, scores candidates using a formalized rubric, and returns a ranked table with evidence and outreach drafts.

---

## How to Use

### Option 1 - Copilot CLI + WorkIQ

Best for: individual use, ad-hoc expert searches with DevOps and transcript signals.

1. Start a Copilot CLI session with WorkIQ enabled
2. Load the agent instructions from `expert-finder-instructions.txt`
3. Query naturally: *"Find experts on container security"*
4. Get ranked results with confidence scores and outreach drafts
5. Say *"deep search"* for parallel multi-signal queries

DevOps signals are available when an Azure DevOps MCP server is configured. Meeting transcript signals require a local transcript store with FTS5 search.

### Option 2 - M365 Declarative Agent (Teams / Copilot)

Best for: team-wide deployment, always-on access inside Teams or M365 Copilot.

1. Edit `appPackage/declarativeAgent.json` - customize instructions for your domain
2. Edit `appPackage/manifest.json` - update app name, description, and IDs
3. Package the `appPackage/` directory into a `.zip`
4. Upload to Teams Admin Center or Copilot Studio
5. Grant required Graph API permissions (see Configuration below)
6. Users access the agent directly in Teams or M365 Copilot

### Option 3 - Portable Skill (Any Platform)

Copy the instructions from `skills.md` into any LLM agent that can query Microsoft Graph APIs. Replace tool-specific references with your platform's equivalents.

---

## Configuration

### Conversation Starters

Pre-configured prompts that appear when users open the agent:

| Starter | Purpose |
|---------|---------|
| "Find experts on [topic]" | General expert search |
| "Who should I contact about [issue]?" | Escalation routing |
| "Who owns [service/project]?" | Ownership lookup |
| "Deep search: [topic] experts" | Parallel multi-signal search |
| "Who can help with [cross-team process]?" | Cross-org discovery |

### Search Modes

| Mode | Queries | Speed | Accuracy | Trigger |
|------|---------|-------|----------|---------|
| **Default** | 1 M365 + 1 DevOps + 1 Transcript | Fast | Good | Automatic |
| **Deep Search** | 4 M365 + 1 DevOps + 1 Transcript | Slower | Best | Say "deep search" |

### Confidence Rubric

| Level | Criteria |
|-------|----------|
| **HIGH** | 3+ signal types, active < 3 months, role aligns. Boosted by 3+ authored docs, DevOps activity, or transcript speaker attribution |
| **MEDIUM** | 2 signal types, active < 6 months, role plausibly related |
| **LOW** | 1 signal type, or activity > 6 months ago |

---

## Dependencies / Setup

### Requirements

- **Node.js 18+** (for DevOps MCP server and transcript store)
- **M365 tenant** with Copilot licensing
- **Azure DevOps** access (optional - for DevOps signals)
- **Meeting transcript store** with FTS5 search (optional - for transcript signals)

### Graph API Permissions

The Declarative Agent requires these Microsoft Graph permissions:

| Permission | Signal |
|------------|--------|
| `People.Read` | People profiles |
| `Mail.Read` | Email threads |
| `Calendars.Read` | Meeting attendance |
| `Files.Read` | Document authorship |
| `ChannelMessage.Read.All` | Teams messages |

### Declarative Agent Capabilities (v1.6)

| Capability | Signal Type | Purpose |
|---|---|---|
| `OneDriveAndSharePoint` | Documents | SharePoint/OneDrive doc search + authorship signals |
| `People` | Profiles | People profiles with related content |
| `Email` | Email | Email thread collaboration patterns |
| `Meetings` | Meetings | Meeting attendance and presentations |
| `TeamsMessages` | Teams | Channel posts and chat threads |
| `WebSearch` | Fallback | External context when M365 data is insufficient |
| `GraphConnectors` | External | Connectors for third-party tools (Jira, ServiceNow, etc.) |

---

## Project Structure

```
Expert Finder/
├── appPackage/
│   ├── manifest.json               # Teams app manifest (v1.19)
│   ├── declarativeAgent.json        # Agent definition + instructions (v1.6)
│   ├── color.png                    # App icon
│   └── outline.png                  # App outline icon
├── expert-finder-instructions.txt   # System prompt (reference copy)
├── skills.md                        # Portable skill template for other teams
├── Expert Finder - Detailed Build Report.md
├── Expert Finder - Feature Launch.md
├── appPackage.zip                   # Pre-built deployment package
└── README.md
```

---

## Development History

7 iterations, each solving a specific speed–accuracy tradeoff:

| Version | Approach | Result |
|---------|----------|--------|
| v1 | Single query baseline | ✅ Strong initial results |
| v2 | 3 parallel queries + validation | ✅ Most accurate, but slow |
| v3 | Single combined query | ❌ Fast but diluted accuracy |
| v4 | Parallel + conditional validation | ✅ Accuracy restored, better speed |
| v5 | Fast default + deep search option | ✅ Best speed–accuracy balance |
| v6 | Declarative Agent with granular v1.6 capabilities | ✅ Teams deployment ready |
| v7 | Meeting transcript integration (speaker attribution) | ✅ 7th signal type added |

---

## Status

**Active - v1.0** · Both deployment modes (CLI + Declarative Agent) are live and working.

Planned: MCP Server packaging to expose Expert Finder as a reusable tool for any MCP-compatible client.

---

## License

MIT
