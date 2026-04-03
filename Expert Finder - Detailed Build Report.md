# Expert Finder Agent — Detailed Build Report & Next Steps

**Author:** Salman Khan | **Date:** March 6, 2026

---

## 1. Project Overview

### Problem
Finding the right internal Microsoft expert for a specific topic is time-consuming. PMs typically rely on tribal knowledge, Teams searches, or asking colleagues who might know someone. This doesn't scale, doesn't work for new team members, and produces inconsistent results.

### Solution
We built an Expert Finder agent that queries Microsoft 365 data (people profiles, emails, meetings, documents) through WorkIQ to return ranked expert recommendations with confidence scores, evidence links, and ready-to-send outreach messages.

### Scope
Primary focus: Gaming Security & Anti-Cheat PM workflows. The agent is domain-flexible and works for any topic.

---

## 2. What We Built — Step by Step

### Phase 1: Initial Agent (v1)
**What:** Created base agent instructions in `expert-finder-instructions.txt` with:
- Goal definition (find experts using WorkIQ signals)
- Response format (name, role, team, evidence, confidence, outreach)
- Behavior rules (don't invent people, ask clarifying questions, prefer precision)
- Compliance rules (respect data boundaries, use approved sources)

**Test:** Ran "WNS expert" query. Got strong results from a single WorkIQ call — 6 ranked candidates with source links.

### Phase 2: Identified 10 Improvements
Analyzed v1 output quality and identified these improvement areas:
1. Multi-signal cross-referencing
2. Recency weighting
3. Org chart validation
4. "Who NOT to contact" signal
5. Formalized confidence scoring
6. Caching/knowledge base
7. Auto-generated outreach drafts
8. Escalation path visualization
9. GitHub/ADO integration
10. Deep search option

**Selected 5 to implement:** #1, #2, #3, #5, #7 (highest accuracy impact).

### Phase 3: Multi-Signal Implementation (v2)
**What:** Added 5 improvements to instructions:
- **Multi-signal search:** 3+ parallel WorkIQ queries (profiles, emails, meetings/docs)
- **Recency weighting:** 4-tier labeling (Currently Active → Likely Stale)
- **Org chart validation:** Follow-up query to verify current role/team
- **Confidence scoring:** Formalized rubric (High=3+ signals, Medium=2, Low=1)
- **Auto outreach drafts:** Always generate for top 2 candidates

**Test:** Ran "SoundWire audio drivers" query. Fired 3 parallel queries + 1 validation query (4 total). Results were significantly more accurate — found Saurin Shah as the explicit SoundWire owner, verified all 5 candidates' current roles.

**Issue:** 4 queries = noticeably slower response time.

### Phase 4: Speed Optimization Attempt (v3)
**What:** Collapsed 3 signal queries into 1 combined comprehensive query. Target: 1 query instead of 4.

**Test:** Ran "Armoury Crate" query. Single query returned faster but results were significantly less accurate — WorkIQ couldn't search as deeply across all signal types simultaneously.

**Learning:** Combined queries dilute search depth per signal type. Speed gain not worth accuracy loss.

### Phase 5: Parallel Queries Fix (v4)
**What:** Reverted to 3 focused queries but ran them IN PARALLEL (simultaneously, not sequentially). Added conditional validation (only if stale signals detected).

**Test:** Ran "Armoury Crate" query again. 3 parallel queries returned rich results — 4 distinct ASUS escalation threads surfaced, correct SAC feature owner identified, ASUS-side contacts included.

**Result:** Accuracy restored to v2 levels with better speed than sequential approach.

### Phase 6: Fast Default + Deep Search (v5 — Final)
**What:** Made single comprehensive query the default for speed. Added "deep search" as user-triggered option that runs 3 parallel queries.

**Test:** Ran "VPN experts" query. Fast single query returned 4 solid candidates. Offer to deep search displayed at bottom.

**Result:** Best balance of speed (30% weight) and accuracy (70% weight).

### Phase 7: M365 Declarative Agent
**What:** Scaffolded the agent as an M365 Copilot Declarative Agent:
- `appPackage/manifest.json` — Teams app manifest (v1.19)
- `appPackage/declarativeAgent.json` — Agent manifest (v1.6 schema) with full instructions, Microsoft Graph capabilities (People, Mail, Calendar, Files), and 6 conversation starters

**Status:** Ready for deployment via Teams Toolkit or Copilot Studio.

---

## 3. Architecture

```
User Query
    │
    ▼
┌─────────────────────────────┐
│  Expert Finder Agent        │
│  (Instructions/System Prompt)│
├─────────────────────────────┤
│  Default: 1 comprehensive   │
│  Deep: 3 parallel queries   │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  WorkIQ / Microsoft Graph   │
│  ┌──────┐ ┌─────┐ ┌──────┐ │
│  │People│ │Email│ │Meet/ │ │
│  │Profiles│ │Threads│ │Docs │ │
│  └──────┘ └─────┘ └──────┘ │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Cross-Reference & Rank     │
│  - Signal count (1-4 types) │
│  - Recency (4 tiers)        │
│  - Role alignment           │
│  - Confidence (H/M/L)       │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Output                     │
│  - Ranked candidates        │
│  - Signal breakdown         │
│  - Source links             │
│  - Outreach drafts (top 2)  │
│  - Deep search offer        │
└─────────────────────────────┘
```

---

## 4. Files Created

| File | Purpose |
|------|---------|
| `expert-finder-instructions.txt` | Agent system prompt / instructions (versioned through 5 iterations) |
| `appPackage/manifest.json` | Teams app manifest for M365 deployment |
| `appPackage/declarativeAgent.json` | Declarative agent manifest with full instructions & capabilities |

---

## 5. Test Results Summary

| Query | Mode | Candidates | Top Confidence | Key Finding |
|-------|------|-----------|----------------|-------------|
| WNS expert | v1 (single query) | 6 | HIGH (Stacey Hanson) | Strong profile + email + meeting signals |
| SoundWire audio drivers | v2 (3 parallel + validation) | 5 | HIGH (Saurin Shah) | Explicit SoundWire profile owner, all roles verified |
| Armoury Crate | v3 (single combined) | 7 | MEDIUM | Results less accurate — diluted signal depth |
| Armoury Crate | v4 (3 parallel) | 5 | HIGH (Kevin Sullivan) | 4 distinct email threads, correct SAC owner found |
| VPN experts | v5 (fast default) | 4 | MEDIUM (Manoj Sharma) | Good speed, solid results, deep search available |

---

## 6. Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Single query default, parallel on demand | 30/70 speed-accuracy balance |
| Conditional org validation (not always) | Avoids unnecessary query when results look fresh |
| Formalized 3-tier confidence rubric | Makes results immediately actionable without second-guessing |
| Auto outreach for top 2 only | More than 2 drafts creates decision fatigue |
| Source links on every result | Trust requires verifiability |

---

## 7. Known Limitations

- **WorkIQ signal gaps:** Some discussions happen in Teams channels that WorkIQ may not fully index. SoundWire conversations, for example, showed minimal email activity.
- **Name ambiguity:** Common names can match multiple people. The agent mitigates this with role/org context.
- **No caching:** Every search is fresh. Repeat queries for the same topic take the same time.
- **Single user's M365 data:** Results are scoped to what the querying user has access to. Different users may get different results for the same query.

---

## 8. Next Steps

### Short-term (next 2 weeks)
1. **Deploy M365 Declarative Agent** to the Gaming Security team for pilot testing
2. **Gather structured feedback** — accuracy rating per search, missing experts, false positives
3. **Add 2-3 more conversation starters** based on team's most common expert-finding needs

### Medium-term (next month)
4. **Add GitHub/ADO code ownership signal** — search CODEOWNERS, recent PRs, and bug assignments for engineering experts
5. **Implement result caching** — store past lookups in a lightweight DB so repeat queries are instant and staleness is tracked
6. **Build "Who NOT to contact" detection** — flag people on leave, recently departed, or who have delegated ownership

### Long-term (next quarter)
7. **MCP Server packaging** — expose Expert Finder as a tool any MCP client can call (VS Code, Claude, other agents)
8. **Escalation path visualization** — auto-generate "Start here → if blocked → exec escalation" decision trees
9. **Cross-team rollout** — generalize beyond Gaming Security for broader Windows org use
10. **Feedback loop** — track which recommended experts actually responded and were helpful, to improve future rankings
