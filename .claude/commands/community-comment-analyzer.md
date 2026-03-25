# Community Comment Analyzer — Make Observability

## Purpose

Collect and analyze community feedback from community.make.com and Reddit about Make Observability features (Make Grid, Execution Logs, Analytics Dashboard, Alerting). Classify comments by feature, sentiment-score them, auto-prioritize based on active product priorities, and deliver a structured HTML digest. Also generates the Make scenario blueprint for automated weekly delivery every Monday at 9:30 AM.

**Trigger this skill when the user asks to:**
- Monitor Make community feedback
- Summarize comments or community posts
- Send or set up a community digest
- Set up weekly community reports
- Track feature friction points in the Make community
- Scrape or summarize community.make.com or Reddit for observability features
- "Check what people are saying about observability"
- "Any community feedback this week?"

---

## Instructions for Claude

### Step 0 — Load Context

Before doing anything, load the following context files to ground your analysis:

- `context/feature-make-grid-observability.md` — feature status, known gaps, and metrics
- `context/company-context.md` — 2025 strategic priorities
- `context/pm-ways-of-working.md` — OKRs, decision thresholds, review cadences
- `context/data-structure.md` — known data gaps (used to flag if a complaint is already a known gap)

**Active features to monitor** (derived from context):
| Feature | Status | Known Gaps |
|---|---|---|
| Make Grid | Open Beta / GA | Scalability 200+ scenarios, no AI suggestions, no sub-org view |
| Execution Logs | GA | No native replay UI for all plans, limited cross-scenario search |
| Analytics Dashboard | GA | No team-level breakdown for free plans, limited export |
| Alerting & Error Monitoring | Partial | No Slack/webhook alerting, no anomaly detection, no SLA alerts |

**2025 Strategic Priorities (for auto-prioritization):**
1. AI Agents & Visual Orchestration
2. Observability at Scale
3. Enterprise Readiness
4. Developer Ecosystem

**OKR Targets (for severity calibration):**
- Make Grid: 40% of 10+ scenario orgs use it ≥1x/month
- Error resolution: Median < 8 min (currently ~18 min)
- Support tickets about debugging/visibility: −25% QoQ
- Alert latency: 80% of errored scenarios trigger alert within 5 min

---

### Step 1 — Collect Community Comments

Use `WebFetch` and `WebSearch` to retrieve recent posts and comments from:

**Primary sources:**
- `https://community.make.com` — search for threads mentioning: "Make Grid", "execution logs", "analytics", "alerting", "error monitoring", "observability", "debugging", "visibility"
- `https://www.reddit.com/r/makerops` and `https://www.reddit.com/r/integromat` — same keyword set

**Search queries to run (adapt as needed):**
```
site:community.make.com "Make Grid" OR "execution log" OR "alerting" OR "analytics dashboard"
site:reddit.com/r/makerops observability OR debugging OR "error log" OR "Make Grid"
site:reddit.com/r/integromat monitoring OR alerting OR "execution log"
```

**Per comment/post, extract:**
- URL / thread title
- Author (anonymized as "User A", "User B" etc.)
- Date posted
- Full text of the comment or post body (truncate at 500 chars if very long)
- Source (community.make.com / Reddit)
- Upvotes / reactions if available

Collect a minimum of **10–30 comments** across sources. If a source is unavailable, note it and proceed with what's available.

---

### Step 2 — Classify Each Comment

For each comment, assign:

**Feature Tag** (one or more):
- `make-grid` — visual dependency map, scenario health, live view
- `execution-logs` — step-level logs, input/output inspection, run replay
- `analytics-dashboard` — consumption, error rates, run volume, credits
- `alerting` — failure notifications, Slack/webhook, anomaly detection
- `general-observability` — cross-feature visibility / debugging / confidence
- `other` — not observability-related

**Comment Type:**
- `bug-report` — something is broken
- `feature-request` — asking for something new
- `frustration` — expressing pain without a specific ask
- `praise` — positive experience
- `question` — asking how to do something (signals discoverability gaps)
- `workaround` — user describing a workaround (signals a gap)

**Sentiment Score:** Integer from −3 to +3
- +3 = very positive (love it, solved my problem)
- +2 = positive (useful, working well)
- +1 = mildly positive
- 0 = neutral / informational
- −1 = mildly negative (minor friction)
- −2 = negative (clear pain, workflow impact)
- −3 = very negative (blocking, strong frustration, considering alternatives)

**Is Known Gap?** (Y / N / Partial)
Cross-reference with `feature-make-grid-observability.md` known gaps and `data-structure.md` data gaps. Mark Y if the complaint maps to a documented gap, Partial if related, N if novel.

**Priority Tag** (auto-assigned based on strategic fit):
- `P1 — Strategic Fit` — maps to a 2025 priority AND an OKR
- `P2 — High Signal` — maps to an OKR but not a top priority
- `P3 — Backlog Signal` — relevant feature feedback but outside current OKRs
- `P4 — Noise` — off-topic, duplicate, or too vague to act on

**P1 Conditions (auto-assign if ANY of these match):**
- Complaint blocks error resolution time metric
- Multiple users mention same gap (3+ comments = signal cluster)
- Gap relates to Alerting (most underdeveloped area)
- Enterprise/team-level pain (supports Enterprise Readiness priority)

---

### Step 3 — Synthesize

**Per-feature summary:**
For each monitored feature, write 2–4 sentences covering:
- Overall sentiment tone (positive / mixed / negative)
- Top recurring theme
- Most critical signal (worst pain or biggest unmet need)
- Whether the feedback aligns with known gaps or reveals something new

**Cross-feature patterns:**
Identify 2–3 recurring themes that span multiple features (e.g., "users consistently mention lack of Slack alerting across Grid and Alerting threads").

**Signal clusters:**
Flag any topic mentioned by 3+ users — these are high-confidence signals, not one-off opinions.

**Weekly trend note:**
If the user has run this skill before, note any shift in sentiment or topic frequency. If this is the first run, note "Baseline established — compare next week."

---

### Step 4 — Priority-Ranked Action List

Produce a ranked list of actionable takeaways for the PM. Format:

| Rank | Priority | Feature | Signal | Comment Count | Avg Sentiment | Known Gap? | Suggested Action |
|---|---|---|---|---|---|---|---|
| 1 | P1 | Alerting | No Slack/webhook alerting | 8 | −2.1 | Y | Fast-track Slack alert MVP — already in backlog |
| 2 | P1 | Make Grid | Performance with 200+ scenarios | 5 | −2.4 | Y | Escalate to Platform Eng — ties to scale OKR |
| ... | ... | ... | ... | ... | ... | ... | ... |

**Suggested action types:**
- "Fast-track — already validated" (known gap + high signal)
- "Add to discovery backlog" (new signal, needs validation)
- "Flag to support team" (recurring question = discoverability gap)
- "No action — monitor" (isolated or ambiguous signal)
- "Escalate to Engineering" (performance/reliability bloat)

---

### Step 5 — Generate HTML Digest

Produce a full, self-contained HTML email digest. It must be copy-pasteable into Gmail or any email client.

**Design spec:**
- Clean, minimal design (white background, #1a1a2e navy headers, #00d4aa teal accents — Make brand adjacent)
- Mobile-responsive (max-width 680px, single column)
- No external dependencies (inline styles only)
- Sections clearly delineated with horizontal rules or background blocks

**Required sections in order:**

1. **Header** — "Make Observability · Community Digest" + date range + "Powered by Claude"
2. **At a Glance** — 4 stat boxes: Total Comments Analyzed, Avg Sentiment Score, Top Feature by Volume, Top Feature by Pain
3. **Priority Alerts** (P1 items only, red-left-border card style) — headline + 1-line context + suggested action
4. **Feature Breakdown** — one collapsible-style section per feature with sentiment bar (emoji-based: 🟢🟡🔴), top themes, and 2–3 selected verbatim quotes
5. **Signal Clusters** — 2–3 cluster cards, each with: cluster topic, comment count, sample quotes
6. **Full Comment Log** — table with: #, Date, Source, Feature Tags, Type, Sentiment, Excerpt (100 chars), URL
7. **Suggested Actions** — the priority-ranked table from Step 4
8. **Footer** — "Next digest: [next Monday at 9:30 AM]" + "Auto-generated by Claude via Make scenario"

**Sentiment bar format (for Feature Breakdown):**
```
🟢🟢🟢🟡🟡🔴  (3 positive, 2 neutral, 1 negative out of 6 total)
```

**Priority Alert card style (inline CSS example):**
```html
<div style="border-left: 4px solid #e74c3c; background: #fff5f5; padding: 12px 16px; margin: 8px 0; border-radius: 4px;">
  <strong style="color: #e74c3c;">⚠ P1 — Alerting</strong><br>
  <span style="color: #333;">8 users mention missing Slack/webhook alerting — avg sentiment −2.1</span><br>
  <em style="color: #666;">Suggested: Fast-track Slack alert MVP — already in backlog</em>
</div>
```

---

### Step 6 — Make Scenario Blueprint (Weekly Automation)

After delivering the digest, output a Make scenario configuration blueprint that the user can implement to automate weekly delivery.

**Scenario name:** `[Claude] Weekly Observability Community Digest`

**Trigger:** Schedule module → Every week, Monday, 09:30 AM (user's local timezone)

**Module chain:**

```
[1] Schedule Trigger
    └─ Every: Monday
    └─ Time: 09:30
    └─ Timezone: (user-configured)

[2] HTTP — GET community.make.com search
    └─ URL: https://community.make.com/search?q=observability+OR+%22Make+Grid%22+OR+%22execution+log%22
    └─ Method: GET
    └─ Parse response: Yes

[3] HTTP — GET Reddit r/makerops
    └─ URL: https://www.reddit.com/r/makerops/search.json?q=observability+OR+debugging&sort=new&limit=25
    └─ Method: GET
    └─ Headers: User-Agent: MakeScenario/1.0

[4] HTTP — GET Reddit r/integromat
    └─ URL: https://www.reddit.com/r/integromat/search.json?q=monitoring+OR+alerting+OR+execution+log&sort=new&limit=25
    └─ Method: GET
    └─ Headers: User-Agent: MakeScenario/1.0

[5] JSON — Parse all responses
    └─ Parse community posts array
    └─ Parse Reddit posts .data.children[].data

[6] HTTP — POST to Claude API (claude-sonnet-4-6)
    └─ URL: https://api.anthropic.com/v1/messages
    └─ Method: POST
    └─ Headers:
        x-api-key: {{ANTHROPIC_API_KEY}}
        anthropic-version: 2023-06-01
        Content-Type: application/json
    └─ Body (JSON):
        {
          "model": "claude-sonnet-4-6",
          "max_tokens": 4096,
          "messages": [{
            "role": "user",
            "content": "You are a PM assistant for Make's Observability team. Analyze the following community comments. For each comment: classify by feature (make-grid, execution-logs, analytics-dashboard, alerting, general-observability), assign type (bug-report, feature-request, frustration, praise, question, workaround), sentiment score (-3 to +3), known gap (Y/N/Partial), and priority (P1/P2/P3/P4). Then produce a full HTML digest following the Make Observability Community Digest format. Comments: {{comments_json}}"
          }]
        }

[7] Text Parser — Extract HTML from Claude response
    └─ Extract content between <html> and </html> tags

[8] Gmail — Send Email
    └─ To: {{USER_EMAIL}}
    └─ Subject: Make Observability Community Digest — Week of {{formatDate(now; 'MMM D, YYYY')}}
    └─ Content: HTML (paste extracted HTML from step 7)
    └─ Content type: HTML

[9] (Optional) Google Sheets — Log run
    └─ Append row: Date | Comments analyzed | P1 count | Avg sentiment
```

**Setup instructions for the user:**
1. In Make, create a new scenario and add the modules above in order
2. Configure the Schedule trigger for Monday 09:30 in your timezone
3. Add your Anthropic API key to Make's Data Stores or Connection manager as `ANTHROPIC_API_KEY`
4. Set `USER_EMAIL` to your Gmail address in the Gmail module
5. Test the scenario manually first (Run once button) before enabling the schedule
6. Enable the scenario — it will run automatically every Monday at 9:30 AM

**API key note:** Store your Anthropic API key in Make's built-in secret store (Data Stores > Connections), never hardcode it in the scenario.

---

## Output Format

Deliver the following in order:

### 1. Collection Summary
> "Collected X comments from community.make.com and Y comments from Reddit between [date range]. Source availability: [notes on any failed fetches]."

---

### 2. Classified Comment Table

| # | Source | Date | Feature Tag | Type | Sentiment | Known Gap | Priority | Excerpt |
|---|---|---|---|---|---|---|---|---|
| 1 | community.make.com | YYYY-MM-DD | make-grid | frustration | −2 | Y | P1 | "The grid just crashes when..." |

---

### 3. Per-Feature Synthesis
[2–4 sentences per feature]

---

### 4. Signal Clusters
[Bullet list of clusters with comment count and summary]

---

### 5. Priority-Ranked Action Table
[As defined in Step 4]

---

### 6. HTML Digest
```html
[Full self-contained HTML email]
```

---

### 7. Make Scenario Blueprint
[Module-by-module configuration as defined in Step 6]

---

## Rules (Always Apply)

- Never fabricate comments — only use content retrieved via WebFetch/WebSearch. If no comments are found, say so clearly.
- Do not include PII — anonymize all usernames.
- Known gaps must be cross-referenced against `feature-make-grid-observability.md` — do not mark a gap as novel if it's already documented.
- P1 assignments must always cite the specific OKR or strategic priority they map to.
- The HTML digest must be fully self-contained with inline styles — no external CSS or JS.
- Always include the Make scenario blueprint, even if the user only asked for the digest.
- If WebSearch or WebFetch is unavailable or rate-limited, note this and offer to run the analysis on user-provided comment text instead.
- Always end with: "Next digest scheduled for [next Monday]. Enable the Make scenario to automate this."
