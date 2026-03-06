# Expert Finder Agent — Team Meeting Summary
## Value Demonstration & AI Tooling Learnings

**Author:** Salman Khan | **Date:** March 6, 2026

---

## What We Built

In a single session (~45 minutes), we built an **AI-powered Expert Finder agent** that searches across Microsoft 365 signals to identify internal experts for any topic. It's designed for PM workflows in Gaming Security & Anti-Cheat but works for any domain.

**Live demo results we ran:**
- **WNS experts** → Found 6 ranked candidates across profiles, emails, and meetings with source links
- **ASUS Armoury Crate** → Surfaced 5 candidates with specific escalation thread context
- **SoundWire audio drivers** → Identified the explicit SoundWire owner (Saurin Shah) plus 4 supporting contacts
- **VPN experts** → Returned 4 candidates spanning engineering, architecture, and support

---

## The Value It Brings

### For PMs
- **Minutes instead of hours:** Finding the right expert used to mean Slack-searching, asking around, and hoping someone knows someone. This agent returns ranked, evidence-backed results in seconds.
- **Confidence you're talking to the right person:** Every recommendation includes which M365 signals contributed (profile, email, meeting, doc) and how recently the person was active.
- **Ready-to-send outreach:** No more staring at a blank email. The agent drafts personalized messages referencing specific evidence to increase response rates.

### For the Team
- **Institutional knowledge that doesn't leave when people do:** The agent finds experts based on live M365 activity, not tribal knowledge.
- **Onboarding accelerator:** New team members can find the right contacts immediately.
- **Repeatable and improvable:** The agent instructions are versioned text — we can tune accuracy and add features iteratively.

---

## What I Learned About AI Tools

### 1. Prompt Engineering is Product Design
We iterated the agent instructions 5 times. Each iteration changed behavior meaningfully:
- **v1:** Single query, basic results
- **v2:** 3 parallel queries — more accurate but 3-4x slower
- **v3:** Single combined query — faster but accuracy dropped
- **v4:** 3 parallel queries with conditional validation — best accuracy, acceptable speed
- **v5 (final):** Fast single query by default + "deep search" option on demand — best of both worlds

**Key insight:** Tuning an AI agent is exactly like tuning a product. You're making speed vs. accuracy tradeoffs, designing user-facing options, and iterating based on real output quality.

### 2. Multi-Signal Cross-Referencing Matters
A single query returns surface-level results. Running focused queries per signal type (profiles, emails, meetings/docs) and cross-referencing produces dramatically richer results. We proved this by A/B testing the Armoury Crate search.

### 3. Confidence Scoring Builds Trust
Adding a formalized rubric (High/Medium/Low based on signal count + recency + role alignment) made results immediately more actionable. Users can quickly decide who to contact first vs. who needs verification.

### 4. Recency and Role Validation Catch Stale Data
People change roles constantly. Labeling candidates as "Currently Active" vs. "Possibly Stale" and validating against current org data prevents wasted outreach to people who've moved on.

### 5. The "Deep Search" Pattern is Reusable
The fast-default + deep-search-on-demand pattern works for any AI agent scenario. Give users speed by default, accuracy when they ask for it.

---

## Deployment Options We Explored

| Option | Effort | Best For |
|--------|--------|----------|
| **Copilot CLI + WorkIQ** (what we used today) | Zero — it's working now | Individual use |
| **M365 Declarative Agent** (scaffolded) | Low — manifest ready to deploy | Team-wide access via Teams/Copilot |
| **MCP Server** | Medium-high | Universal tool access across any AI client |

---

## Next Steps
1. Deploy the M365 Declarative Agent to our team for pilot testing
2. Gather feedback on accuracy and coverage gaps
3. Add GitHub/ADO code ownership signals for engineering expert searches
4. Explore caching past lookups for instant repeat queries
