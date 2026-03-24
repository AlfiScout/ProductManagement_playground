# PM Ways of Working @ Make — Observability Team

> Context for how the Observability PM operates day-to-day.
> Useful reference for AI-assisted PM work: how to frame problems, what artifacts to produce, how decisions are made.

---

## Decision-Making Framework

### Prioritization Model
Make PMs use a combination of:
1. **Impact vs Effort** — rough sizing before detailed scoping
2. **RICE** (Reach × Impact × Confidence ÷ Effort) — for competing roadmap items
3. **Jobs-to-be-Done (JTBD)** — framing features around user outcomes, not feature requests
4. **Business metric alignment** — does this move ARR, retention, activation, or NPS?

### When to Build vs. When to Wait
- Build: Problem is validated by 3+ independent customer signals AND aligns with a strategic theme
- Investigate further: Problem is clear but solution is not
- Defer: Problem is real but affects <5% of users and no strategic fit
- Kill: Low demand, high complexity, no data to validate

---

## Key Artifacts

### 1. Opportunity Brief (pre-PRD)
Used early in discovery. Answers:
- What is the problem? For whom?
- What evidence do we have? (quant + qual)
- What are the potential solutions?
- What would we need to believe for this to be worth building?
- What's the rough size of the opportunity?

### 2. Product Requirements Document (PRD)
Used when moving to design + engineering. Contains:
- Problem statement and user story
- Scope: what's in and explicitly what's out
- Acceptance criteria
- Success metrics (primary + secondary)
- Open questions and dependencies
- Edge cases

### 3. Launch Brief
Used at release time. Contains:
- What shipped and what didn't
- How to measure success (metrics + timeline)
- Rollout plan (% rollout, flags, regions)
- Communication plan (internal, external)
- Known risks

---

## Discovery Practices

### Customer Interview Cadence
- Target: 2–4 customer conversations per week during active discovery
- Formats: 30-min user interviews, async video feedback (Loom), support ticket analysis
- Synthesis: Affinity mapping in Miro or FigJam, weekly summary to team

### Data-First Habits
- Pull usage data before starting any discovery (what do we know already?)
- Define the metric question before querying: "I want to know X because I need to decide Y"
- Never present data without confidence level (sample size, time range, segment)

### Assumption Mapping
Before building, list:
- What must be true for this to succeed?
- Which assumptions are highest risk?
- How can we test them cheaply?

---

## Collaboration Norms

### With Engineering
- Weekly sync: current sprint, blockers, upcoming decisions
- PM writes requirements; engineers write technical design
- Engineers are consulted on feasibility before scope is locked
- No scope changes after sprint start without explicit trade-off discussion

### With Design
- PM and Design do discovery together when possible
- PM owns the "what" and "why"; Design owns the "how" (UX)
- Usability testing is a shared responsibility

### With Data / Analytics
- PM defines what needs to be measured; Data defines how to instrument it
- Event taxonomy proposals go through PM → Data review → Engineering
- Dashboards and queries owned by Data team; PM reviews weekly

### With Sales & Customer Success
- Monthly win/loss review with Sales
- Quarterly roadmap preview to CS leads
- PM joins customer calls on request (especially enterprise accounts)

---

## Metrics & Review Cadence

### Weekly
- Scenario error rate (org-level and global)
- Active scenarios count (week-over-week)
- Make Grid adoption (% orgs with 10+ scenarios that opened Grid)
- Operation consumption vs. limits (% orgs within 80% of limit)

### Monthly
- Feature adoption for shipped observability features (D7, D30 activation)
- Support ticket volume for observability-related issues
- NPS driver analysis (do observability-related themes appear in detractors?)

### Quarterly
- Roadmap review with leadership
- OKR check-in: are we hitting the outcomes we defined?
- Competitive landscape review: what have competitors shipped?

---

## OKR Structure (Example — Observability PM)

### Objective
Users and teams have full visibility and control over their automation health.

### Key Results
- KR1: 40% of orgs with 10+ active scenarios use Make Grid at least once/month (adoption)
- KR2: Median time-to-resolution for scenario errors drops from 18 min to under 8 min (efficiency)
- KR3: Support tickets categorized as "debugging/visibility" decrease by 25% QoQ (support burden)
- KR4: Error notification coverage: 80% of errored scenarios trigger a user-visible alert within 5 min (reliability)

---

## Tools Stack
| Tool | Usage |
|---|---|
| Productboard | Roadmap, feature voting, customer insights |
| Jira | Sprint planning, story tracking |
| Figma | Design collaboration |
| Miro | Discovery workshops, affinity mapping |
| Mixpanel / Amplitude | Product analytics, funnel analysis |
| Datadog (internal) | Infrastructure and event monitoring |
| Notion | Documentation, decision logs |
| Slack | Async comms, stakeholder updates |
| Loom | Async customer interviews, feature walkthroughs |
| Claude / ChatGPT | PRD drafting, synthesis, research acceleration |

---

## Common PM Anti-Patterns to Avoid
- Writing requirements without talking to customers first
- Measuring success only by whether the feature shipped (not whether it was used)
- Treating the roadmap as a commitment rather than a hypothesis
- Adding scope without removing something else
- Building dashboards nobody uses because they weren't tied to a real decision
