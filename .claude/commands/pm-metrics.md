# PM Metric & Observability Designer

## Purpose
Define how to measure a feature's success AND assess the feasibility of actually measuring it — given real data constraints.

Use this when: starting a new feature, writing a PRD, preparing for a planning cycle, or evaluating whether a metric is actually actionable.

---

## Instructions for Claude

The user will describe a feature, initiative, or product area. Your job is to produce a structured measurement framework following the output format below.

**Before generating output:**
- Ask: what is the feature? What problem does it solve? Who is the user?
- If context files are available (company-context.md, data-structure.md, feature files), load them to assess data availability realistically.
- If the user hasn't described the feature clearly, ask one clarifying question before proceeding.

**Guiding principles:**
- Every metric must be tied to a decision. If a metric doesn't help someone decide something, don't include it.
- Be explicit about data uncertainty. Don't assume events exist — check against data-structure.md or flag as unknown.
- Prefer leading indicators where possible (they give you time to act).
- Flag vanity metrics explicitly and explain why they're excluded.
- Be honest about blind spots — what could mislead us even if the metric is green?

---

## Output Format

### Feature Being Measured
[Name of the feature or initiative]

---

### 1. Decision Framework

**What decisions will this metric set enable?**
- [Decision 1 — e.g., "Should we invest more in X or pivot to Y?"]
- [Decision 2 — e.g., "Is adoption sufficient to justify removing the legacy feature?"]
- [Decision 3 — e.g., "Do we need to improve discoverability or the feature itself?"]

**Decision thresholds that matter:**
| Decision | Green (proceed) | Yellow (investigate) | Red (act / stop) |
|---|---|---|---|
| [Decision 1] | [threshold] | [threshold] | [threshold] |
| [Decision 2] | [threshold] | [threshold] | [threshold] |

---

### 2. Success Definition

**What does success look like 30 / 60 / 90 days after launch?**

- **30 days**: [Early signal — is it being discovered and tried?]
- **60 days**: [Adoption signal — is it becoming part of workflow?]
- **90 days**: [Outcome signal — is it solving the problem?]

**The one metric that matters most:** [Single most important indicator of success, and why]

---

### 3. Metrics

#### Critical (decision-driving)
> These directly determine whether we continue, adjust, or stop. Reviewed weekly.

| Metric | Definition | Target | Data Source | Availability |
|---|---|---|---|---|
| [Metric name] | [Precise definition] | [Target value or direction] | [Event / table / tool] | A / P / M |

#### Important (diagnostic)
> These explain *why* the critical metrics are moving. Reviewed bi-weekly.

| Metric | Definition | What it tells us | Data Source | Availability |
|---|---|---|---|---|
| [Metric name] | [Precise definition] | [What behavior this reveals] | [Source] | A / P / M |

#### Supporting (context)
> These provide background. Don't optimize for these. Reviewed monthly.

| Metric | Definition | Data Source | Availability |
|---|---|---|---|
| [Metric name] | [Definition] | [Source] | A / P / M |

*Availability key: A = available now, P = partially available / needs work, M = missing / needs instrumentation*

---

### 4. Data Requirements

**Events needed to power these metrics:**

| Event Name | Trigger | Properties Required | Currently Exists? |
|---|---|---|---|
| `[event.name]` | [When it fires] | [Fields needed] | A / P / M |

**Data points required (beyond events):**
- [Entity field or attribute needed, e.g., "scenario.module_count at time of run"]
- [Aggregation needed, e.g., "7-day rolling error rate per scenario"]

---

### 5. Data Availability Assessment

**Likely available (can measure now):**
- [Metric or data point] — [why it's likely available]

**Unclear (needs verification):**
- [Metric or data point] — [what's uncertain and who to ask]

**Missing (would need new instrumentation):**
- [Metric or data point] — [what doesn't exist yet]

---

### 6. Data Gaps & Requests

**What needs to be tracked that isn't today:**

| Gap | Impact on Measurement | Suggested Instrumentation | Priority |
|---|---|---|---|
| [What's missing] | [Which metric it blocks] | [Where to add the event/field] | High / Med / Low |

**Suggested new events:**
```
Event: [event.name]
Trigger: [when to fire]
Properties:
  - [property_1]: [type] — [description]
  - [property_2]: [type] — [description]
Owner: [Engineering team / Data team]
```

---

### 7. Measurement Risks & Blind Spots

**What might mislead us (even if metrics look good):**
- [Risk 1] — [Why it could give a false positive/negative]
- [Risk 2] — [Confounding factor to watch for]

**What we might miss entirely:**
- [Blind spot 1] — [What this measurement framework can't see]
- [Blind spot 2] — [User behavior that won't show up in the data]

**Goodhart's Law watch:** [Is there a risk that optimizing for our key metric will cause gaming or perverse behavior?]

---

### 8. Leading vs. Lagging Indicators

| Indicator | Type | What It Predicts | Lag Time |
|---|---|---|---|
| [Metric] | Leading | [What it predicts] | [Days/weeks before outcome visible] |
| [Metric] | Lagging | [What outcome it confirms] | [Delay from action to signal] |

**Recommended leading metric to watch first:** [Name + rationale]

---

### 9. Measurement Plan Summary

| Phase | Timeframe | Focus | Review Cadence |
|---|---|---|---|
| Pre-launch | Before ship | Instrument gaps, baseline current state | One-time |
| Early signal | Week 1–2 | Discovery, error rate, initial usage | Daily |
| Adoption | Week 3–6 | Retention, D7/D30 activation, engagement | Weekly |
| Outcome | Month 2–3 | Problem solved? Business impact? | Bi-weekly |

---

## Rules (Always Apply)
- Tie every metric to a decision — if you can't name the decision, drop the metric
- Avoid vanity metrics (page views, feature click counts without context)
- Highlight uncertainty — be explicit when data availability is unknown
- Do not assume perfect data — flag gaps honestly
- Prefer 3–5 metrics over 15 — fewer, better metrics beat comprehensive dashboards
- Never skip the blind spots section — the most valuable part is what you can't see
