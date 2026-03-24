# Feature Context — Make Grid & Observability Suite

> This is the primary feature area owned by the Observability PM at Make.
> Covers: Make Grid, Execution Logs, Scenario Run History, Analytics Dashboard, Alerting & Error Monitoring.

---

## Feature: Make Grid

### What It Is
Make Grid is a live, auto-generated visual map of an organization's entire automation landscape. It shows all active scenarios, their connections to apps, their interdependencies, and the health status of each — in a single centralized view.

### Problem It Solves
As teams scale their automation usage (20, 50, 200+ scenarios), visibility becomes the bottleneck:
- Users don't know which scenarios depend on the same connection
- When one thing breaks (e.g., a Salesforce API change), teams spend hours finding all affected scenarios
- Leadership has no way to review automation health without opening each scenario individually
- Onboarding new team members is painful — no map of "what we've built"

### Target Users
- **Primary**: Automation owners / ops leads managing 10+ scenarios
- **Secondary**: IT managers and team leads who need an overview without managing scenarios directly
- **Tertiary**: New team members onboarding to an existing automation stack

### Core Capabilities (Current / In Progress)
| Capability | Status | Notes |
|---|---|---|
| Live scenario map | GA (Open Beta) | Auto-generated, no manual setup |
| App/connection dependencies | GA | Shows which apps each scenario uses |
| Scenario health indicators | In Progress | Flags scenarios with errors or disabled status |
| Credit/operation consumption layer | In Progress | Shows which scenarios consume most operations |
| Grid snapshot sharing | Planned | Share read-only snapshots for async debugging |
| Drill-down to scenario detail | Planned | Click node → open scenario or execution log |

### Key Metrics to Track
- % of orgs with 10+ scenarios that have viewed Make Grid at least once (adoption)
- Time-to-resolution for cross-scenario errors (before/after Grid)
- Support tickets related to "broken dependency" or "I don't know which scenarios are affected" (reduction)
- Grid views per org per week (engagement)

### Known Gaps / Open Questions
- Does the map scale visually for orgs with 200+ scenarios? (performance + UX)
- Should AI suggest which scenarios are "at risk" based on dependency graph? (future)
- How do we handle sub-organizations / team-level views?

---

## Feature: Execution Logs

### What It Is
A detailed, timestamped log of every module execution within a scenario run. Shows input data, output data, duration, status (success/error), and any error messages.

### Problem It Solves
- Debugging automation failures is time-consuming without full data visibility
- Users need to understand what data was processed at each step to diagnose issues
- Data loss risks when a scenario partially executes

### Current Capabilities
- Step-by-step module input/output inspection
- Error message surfacing with error code
- Duration per module
- Filter by status (success, error, warning)
- Scenario run history (configurable retention)

### Key User Actions (Jobs to Be Done)
1. "I need to find out why this scenario failed yesterday at 3pm"
2. "I need to see what data was passed to the third-party API"
3. "I need to replay this run with the same data after fixing the scenario"

### Scenario Run Replay (2025 Feature)
- Re-run a scenario using historical trigger data
- Useful for: fixing errors and reprocessing, validating logic changes, backfilling data
- Users can label runs with custom names for easier tracking

---

## Feature: Analytics Dashboard

### What It Is
A reporting layer showing scenario performance, operation consumption, error rates, and team activity over time.

### Core Metrics Surfaced
- Operations consumed (by scenario, team, time period)
- Run volume (number of scenario executions)
- Error rate (% of runs with errors)
- Top scenarios by consumption
- Credit budget utilization

### Audience
- Team leads / ops managers tracking efficiency and budget
- Admins managing org-level consumption

---

## Feature: Alerting & Error Monitoring

### What It Is
Notification system that alerts users when scenarios fail, hit error thresholds, or consume unexpected amounts of operations.

### Current State
- Email notifications on scenario failure
- In-app error count badges
- Error handlers (custom fallback logic within scenarios)

### Gap / Opportunity
- No proactive alerting on anomalous operation consumption spikes
- No Slack/webhook-based alerting natively (users workaround with self-built notification scenarios)
- No SLA-style alerting (e.g., "alert me if this scenario hasn't run in 2 hours")

---

## Observability Vision (North Star)
Users and teams should be able to answer these questions in under 60 seconds — without opening any individual scenario:

1. Is everything running as expected right now?
2. What broke, when, and why?
3. Which automations are most at risk or consuming the most resources?
4. What changed recently that might have caused this issue?

---

## Related Roadmap Themes
- **AI-powered diagnostics**: "This scenario has failed 3 times with the same error. The likely cause is X. Suggested fix: Y."
- **Cross-scenario impact analysis**: "If this connection breaks, these 14 scenarios will be affected."
- **Consumption forecasting**: "At current growth, you'll hit your operation limit in 12 days."
- **Audit trail for enterprise**: Who changed what, when — for compliance and debugging.

---

## Dependencies & Stakeholders
| Team | Dependency |
|---|---|
| Data Infrastructure | Event pipeline for execution data |
| Platform Engineering | Scenario execution engine, log storage |
| AI Team | AIOps features, anomaly detection |
| Enterprise/Sales | Compliance, audit trail requirements |
| Design | Grid UX, log readability, alerting UI |
