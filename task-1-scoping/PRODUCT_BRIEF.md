# Product Brief: Marketing Performance Intelligence Tool
**Version:** 1.0 (Scoping Document)  
**Author:Sai Priya N 
**Date:** 29 May 2026  
**Status:** Draft for Review

---

## 1. The Problem

The marketing team at a multi-client marketing technology company currently answers one question more than any other:

> *"How is our marketing performing across channels right now, and where should we be focusing?"*

Today, answering this question looks like this:
- Someone manually logs into Google Ads, Meta Ads Manager, and possibly other tools
- They pull numbers from each, copy them into a spreadsheet or a Slack message
- They stitch together a narrative and send it off
- The answer is inconsistent — it depends on who did it, when, and what they remembered to check
- If that person is busy, the question goes unanswered

This is a knowledge bottleneck. It slows down decisions, creates inconsistency, and makes the team dependent on one or two people.

**The goal is to eliminate this bottleneck** — not by replacing human judgment, but by making the underlying data instantly accessible and consistently presented to anyone who needs it.

---

## 2. Who Is This For?

**Primary User: The Internal Marketing Analyst / Account Manager**

This is someone on the Tacheon/Smacient team who manages one or more client brands. They need to answer performance questions quickly — for themselves, or to respond to a client.

They are NOT data engineers. They should not need to write SQL or build reports. They need fast, trustworthy answers in plain language.

**Secondary User: The Internal Team Lead / Manager**

Someone who wants a quick overview across all clients — not to dig into details, but to spot what needs attention.

**Not in scope for v1: The Client**

Clients might eventually get access, but building for both internal users and clients in v1 doubles the complexity (different trust levels, different data sensitivity, different UI needs). We start internal.

---

## 3. What the Tool Does (v1 Scope)

### The Core Interaction

A user opens the tool, selects a client brand and a time period, and sees:

1. **A performance snapshot** — key metrics for each active channel (e.g. spend, impressions, clicks, conversions, cost per result) for the selected period
2. **A trend indicator** — is each metric going up, down, or flat compared to the previous period?
3. **A plain-language summary** — 3 to 5 bullet points that explain what the data is saying and flag anything that needs attention

That's it. No building custom reports. No dragging and dropping widgets. No pivot tables.

### What a Successful Interaction Looks Like

An account manager gets a message from a client asking how their paid social campaign is doing. Instead of spending 20 minutes pulling data, they open the tool, select the client, set the date range to "last 7 days," and within 30 seconds they have a clear answer. They can respond to the client in 2 minutes instead of 20.

### Key Features in v1

| Feature | Description |
|---|---|
| Client selector | Choose which brand/client to view |
| Date range picker | Last 7 days, last 30 days, custom range |
| Channel performance cards | One card per channel (e.g. Google Ads, Meta Ads) showing core metrics |
| Period-over-period change | Show % change vs previous equivalent period |
| Plain-language summary | Auto-generated bullets summarising performance and flagging anomalies |
| Data freshness indicator | Show when data was last updated so users know how current it is |

---

## 4. What the Tool Does NOT Do (v1)

Being explicit about what is out of scope is just as important as defining what is in scope.

| Out of Scope | Reason |
|---|---|
| Client-facing access | Different trust model, auth complexity, UI needs — phase 2 |
| Custom report builder | Adds massive complexity; not what users actually need day-to-day |
| Recommendations engine | AI-generated strategic advice requires validation and trust-building first |
| Budget pacing / forecasting | Useful but distinct problem; tackle after core performance view is solid |
| Social listening / organic metrics | Requires different data sources; keep v1 focused on paid channels |
| Automated alerts / notifications | Valuable but adds infrastructure complexity; log it for v2 |
| Multi-channel attribution modelling | Complex, contested, and requires more data history than v1 needs |

The principle: **v1 should make the existing workflow 10x faster, not replace it entirely.**

---

## 5. Data: Where It Comes From and How It Gets There

### Data Sources (Paid Channels, v1)

- **Google Ads API** — campaign spend, impressions, clicks, conversions
- **Meta Ads API (Facebook/Instagram)** — same core metrics
- Optionally: LinkedIn Ads, TikTok Ads (if clients are active there)

### How Data Gets In

The tool does NOT ask the team to change their existing tools. Instead:

- A backend pipeline (scheduled, e.g. nightly or every 6 hours) calls each platform's API and pulls the latest performance data
- Data is stored in a central data warehouse (e.g. BigQuery)
- The tool reads from this warehouse — it does not call platform APIs in real time

This approach means:
- The tool is fast (reads from a database, not a live API)
- Data is consistent regardless of who is viewing
- If an API is temporarily down, the tool still shows the last available data with a clear "last updated" timestamp

### Data Reliability and Trust

Users will only trust what the tool shows if they understand where it comes from and how fresh it is. Every data card in the tool will show:
- The data source (e.g. "From Google Ads API")
- When it was last synced (e.g. "Updated 3 hours ago")

If a sync failed, the card clearly says so rather than showing stale data silently.

---

## 6. How a User Interacts With It

The tool is a **simple internal web dashboard** — accessible via browser, no installation required.

**Flow:**
1. User logs in (simple internal auth — no need for SSO in v1)
2. Selects a client from a dropdown
3. Selects a date range
4. Sees a dashboard with one card per active channel
5. Reads the plain-language summary at the top
6. Done

No training required. No manual. If you need a manual, the tool is too complicated.

---

## 7. What Would Make Users Trust It

- **Transparent data sources** — every number links back to where it came from
- **Clear data freshness** — always show when data was last updated
- **Consistency** — same question, same inputs, same answer every time
- **Honest gaps** — if data is missing or a sync failed, say so clearly; never show a zero when the real answer is "we don't know"
- **No black boxes** — the plain-language summary explains what it's saying, not just what the conclusion is

---

## 8. Technical Approach (High Level)

| Component | Choice | Reason |
|---|---|---|
| Data warehouse | BigQuery (GCP) | Already used by the team |
| Data pipeline | Python scripts, scheduled via Cloud Scheduler or similar | Simple, maintainable, observable |
| Backend | Lightweight Python API (e.g. FastAPI) | Reads from BigQuery, serves the frontend |
| Frontend | Simple React dashboard | Familiar, fast to build, easy to iterate |
| Auth | Basic internal login | No need for complex SSO in v1 |
| Hosting | GCP Cloud Run or similar | Consistent with existing stack |

The key constraint: **the team does not change the tools they use.** This tool fits around their existing stack — it does not require them to adopt new platforms or change how they work in their existing tools.

---

## 9. What I Would Revisit With More Time

- **Speak to actual users first.** I made assumptions about what the primary user needs. Ideally, I'd spend 30 minutes interviewing two or three account managers before writing a single line of this brief. Their version of "what's most useful" might surprise me.
- **Validate the data sources.** I assumed Google Ads and Meta Ads are the main channels. In reality, the mix varies by client. A discovery session with the team would clarify this.
- **Define the plain-language summary more precisely.** "AI-generated bullets" sounds simple but requires careful design — what triggers a flag? What language is appropriate? This needs iteration with real users.
- **Consider a Slack-first interaction.** Many internal teams actually want answers in Slack, not a separate tool. A Slack bot that answers "how is [client] performing this week?" might solve the same problem with less friction than a full dashboard. Worth testing both.
- **Build a lightweight prototype and test it with one real user** before committing to the full build.

---

## 10. Definition of Done for v1

v1 is complete when:
- An internal team member can select any client and date range and see accurate, up-to-date performance data for their active paid channels
- The data is refreshed automatically at least once every 24 hours
- The plain-language summary correctly identifies the top 1–2 things worth noting in the data
- A new team member can use the tool without any training or documentation
- The team no longer needs to manually pull data to answer the standard performance question

---

*This brief was written as part of a product scoping exercise. It represents a first-pass definition intended to provoke discussion and refinement, not a final specification.*
