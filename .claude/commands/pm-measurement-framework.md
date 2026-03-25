# PM Feature Measurement Framework

## Purpose
Define how to measure a feature's success AND assess the feasibility of actually measuring it — given real-world data constraints. Goes beyond metric selection to classify the feature type, surface guardrail risks, and produce an actionable pre/post-launch plan.

Use this when: writing a PRD, starting a new feature, preparing for a planning cycle, or stress-testing whether a metric is actually actionable.

---

## Instructions for Claude

The user will describe a feature, initiative, or product area. Produce a structured measurement framework following the output format below.

**Before generating output:**
- Identify: what is the feature, what problem does it solve, who is the user?
- If context files are available (`company-context.md`, `data-structure.md`, feature files), load them to assess data availability realistically and align metrics with current OKRs.
- If the feature type is ambiguous, infer it from the description — do not ask unless the input is genuinely unclear.

**Guiding principles:**
- Every critical metric must be tied to a decision. If it doesn't drive action, downgrade or drop it.
- Be explicit about data uncertainty. Don't assume events exist — check against `data-structure.md` or flag as unknown.
- Prefer leading indicators where possible — they give time to act.
- Flag vanity metrics and explain why they're excluded.
- Guardrail metrics are not optional — every feature has failure modes worth watching.

## Mode
Choose the mode BEFORE generating output.
### Use QUICK MODE when:
- Feature is early-stage or loosely defined
- User provides minimal context (1–3 sentences)
- Feature is small UX/UI change
- Goal is fast exploration, not full validation

---

### Use FULL MODE when:
- Feature is well-defined or strategic
- User asks for PRD-level or detailed evaluation
- There is clear problem statement and scope
- Decisions require deeper validation

---

### Default Behavior:
- If uncertain → use QUICK MODE
- Do NOT ask the user which mode to use unless explicitly requested

## Output Format

### Feature Being Measured
[Name of the feature or initiative]

---

### 1. Feature Classification

**Type:** Adoption / Reliability / Efficiency / Monetization / Mixed

**Why this classification matters for measurement:**
[1–2 sentences on how the feature type shapes what to measure — e.g., Reliability features need error rate and SLA metrics before adoption metrics; Monetization features need revenue attribution, not just engagement]

---

### 2. Decision Framework

**What decisions will this metric set enable?**
- [Decision 1 — e.g., "Should we invest more in X or pivot to Y?"]
- [Decision 2 — e.g., "Is adoption sufficient to justify removing the legacy feature?"]
- [Decision 3 — e.g., "Do we need to improve discoverability or the feature itself?"]

**Decision thresholds:**
| Decision | 🟢 Green (proceed) | 🟡 Yellow (investigate) | 🔴 Red (act / stop) |
|---|---|---|---|
| [Decision 1] | [threshold] | [threshold] | [threshold] |
| [Decision 2] | [threshold] | [threshold] | [threshold] |

---

### 3. Success Definition

**User perspective — what does success feel like for them?**
[What the user can now do, faster, more reliably, or with more confidence — in plain language]

**Business/product perspective — what outcome does it drive?**
[Retention, activation, error reduction, revenue signal, etc. Tie to current OKRs if context is available]

**30 / 60 / 90 days after launch:**
- **30 days:** [Early signal — is it being discovered and tried?]
- **60 days:** [Adoption signal — is it becoming part of workflow?]
- **90 days:** [Outcome signal — is it solving the problem?]

**The one metric that matters most:** [Single most important indicator of success, and why]

---

### 4. Metrics

#### 🔴 Critical (Decision-Driving)
> Directly determine whether we continue, adjust, or stop. Reviewed weekly.

| Metric | Definition | Target | Data Source | Availability |
|---|---|---|---|---|
| [Metric name] | [Precise definition] | [Target value or direction] | [Event / table / tool] | A / P / M |

#### 🟡 Important (Diagnostic)
> Explain *why* critical metrics are moving. Help identify root causes. Reviewed bi-weekly.

| Metric | Definition | What it tells us | Data Source | Availability |
|---|---|---|---|---|
| [Metric name] | [Precise definition] | [What behavior this reveals] | [Source] | A / P / M |

#### ⚪ Supporting (Contextual)
> Background context. Don't optimize for these. Reviewed monthly.

| Metric | Definition | Data Source | Availability |
|---|---|---|---|
| [Metric name] | [Definition] | [Source] | A / P / M |

*Availability key: A = available now, P = partially available / needs work, M = missing / needs instrumentation*

---

### 5. Leading vs. Lagging Indicators

| Indicator | Type | What It Predicts / Confirms | Lag Time |
|---|---|---|---|
| [Metric] | Leading | [What early signal this gives] | [Days/weeks before outcome visible] |
| [Metric] | Lagging | [What outcome it confirms] | [Delay from action to signal] |

**Recommended leading metric to watch first:** [Name + rationale]

---

### 6. Data Requirements

**Events needed to power these metrics:**

| Event Name | Trigger | Properties Required | Currently Exists? |
|---|---|---|---|
| `[event.name]` | [When it fires] | [Fields needed] | A / P / M |

**Data points required beyond events:**
- [Entity field or attribute — e.g., "scenario.module_count at time of run"]
- [Aggregation needed — e.g., "7-day rolling error rate per scenario"]

---

### 7. Data Availability Assessment

**Likely available — can measure now:** (Confidence: High)
- [Metric or data point] — [why it's likely available]

**Unclear — needs validation:** (Confidence: Medium)
- [Metric or data point] — [what's uncertain and who to ask]

**Missing — needs new instrumentation:** (Confidence: Low)
- [Metric or data point] — [what doesn't exist yet]

---

### 8. Data Gaps & Instrumentation Needs

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
Dependencies: [Data team / Engineering team]
```

---

### 9. Measurement Risks & Blind Spots

**What might mislead us (even if metrics look good):**
- [Risk 1] — [why it could give a false positive/negative]
- [Risk 2] — [confounding factor to watch for]

**What we might miss entirely:**
- [Blind spot 1] — [what this framework can't see]
- [Blind spot 2] — [user behavior that won't show up in the data]

**Goodhart's Law watch:** [Is there a risk that optimizing for our key metric causes gaming or perverse behavior?]

---

### 10. Guardrail Metrics (Side Effects)

> These are not success metrics — they're failure detectors. If any of these move in the wrong direction, investigate immediately regardless of how good the primary metrics look.

| Guardrail | What it detects | Threshold to investigate | Data Source |
|---|---|---|---|
| [e.g., Scenario run error rate] | [Feature increased failure rate as side effect] | [e.g., >2% increase week-over-week] | [Source] |
| [e.g., Page load / response time] | [Performance degradation] | [e.g., p95 latency increases >200ms] | [Source] |
| [e.g., Support ticket volume] | [User confusion or broken experience] | [e.g., >10% increase in related ticket category] | [Source] |

**Highest-risk side effect to watch:** [Name the most likely unintended consequence and why]

---

### 11. Measurement Plan Summary

| Phase | Timeframe | Focus | Review Cadence |
|---|---|---|---|
| Pre-launch | Before ship | Instrument gaps, establish baselines | One-time |
| Early signal | Week 1–2 | Discovery, error rate, initial usage | Daily |
| Adoption | Week 3–6 | Retention, D7/D30 activation, engagement depth | Weekly |
| Outcome | Month 2–3 | Problem solved? Business impact? Guardrails stable? | Bi-weekly |

### 12. Measurement Cost & Complexity

- Low / Medium / High effort
- Dependencies required
- Is it worth the investment?
---

## Rules (Always Apply)
- Tie every critical metric to a decision — if you can't name the decision, downgrade or drop it
- Avoid vanity metrics (raw page views, feature click counts without context)
- Highlight uncertainty — be explicit when data availability is unknown or assumed
- Do not assume perfect data — flag gaps honestly
- Prefer 3–5 metrics over 15 — fewer, better metrics beat exhaustive dashboards
- Never skip Guardrail Metrics — every feature has failure modes
- Never skip Blind Spots — the most valuable section is what the framework can't see
- State assumptions clearly when unsure
