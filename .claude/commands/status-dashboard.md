# Feature Status Dashboard — Executive Summary

## Purpose

Pull live feature status from Jira, Monday.com, and a local prioritization file. Synthesize into a top-priority executive briefing — newsletter-quality HTML formatted in Make brand colors — suitable for C-suite sharing. Show the draft to the user for approval, then send it by email as if sent directly by the PM.

**Trigger this skill when the user asks to:**
- Get a feature status summary or dashboard
- Prepare an exec update, exec briefing, or leadership report
- Send a status email to executives or leadership
- Show what's shipping, what's at risk, or what's top priority
- "Give me the exec summary"
- "What's the status of my top features?"
- "Prepare the weekly leadership update"
- "Send the exec email"

---

## Instructions for Claude

### Step 0 — Load Context

Before fetching any data, load these files to ground the analysis:

- `context/feature-make-grid-observability.md` — owned features, statuses, known gaps, OKR targets
- `context/company-context.md` — strategic priorities, competitive context, business model
- `context/pm-ways-of-working.md` — OKR structure, decision thresholds, review cadences
- `context/data-structure.md` — known data gaps (used to flag metric risks)

**Owned features (for reference — override with live data once fetched):**
| Feature | Area | North Star |
|---|---|---|
| Make Grid | Observability | 40% of 10+ scenario orgs use it ≥1×/month |
| Execution Logs + Run Replay | Observability | Median error resolution < 8 min |
| Analytics Dashboard | Observability | Track credit burn + run volume at team level |
| Alerting & Error Monitoring | Observability | 80% errored scenarios trigger alert within 5 min |

---

### Step 1 — Locate the Local Prioritization File

Check for a local prioritization file in these locations (in order):

1. `prioritization.csv`
2. `prioritization.json`
3. `roadmap.csv`
4. `roadmap.json`
5. `features.csv`
6. `features.md`
7. Any file in the working directory matching `*prioriti*`, `*roadmap*`, or `*backlog*`

If found: read it and extract feature names, priority scores/tiers, status, owner, and any notes.

If NOT found: tell the user you couldn't find a local prioritization file and ask them to either:
- Provide the file path, or
- Paste the data inline (accepts CSV, JSON, or plain text table), or
- Confirm to proceed with Jira + Monday.com data only

---

### Step 2 — Fetch Data from Jira

Use `WebFetch` to call the Jira REST API. Ask the user for their Jira credentials if not already provided (base URL, project key, and API token).

**Required config (prompt user if missing):**
- `JIRA_BASE_URL` — e.g., `https://yourcompany.atlassian.net`
- `JIRA_PROJECT_KEY` — e.g., `OBS` or `GRID`
- `JIRA_API_TOKEN` — Personal Access Token or Basic Auth base64
- `JIRA_EMAIL` — account email (for Basic Auth)

**API calls to make:**

**A. In-progress epics/stories (current sprint or active items):**
```
GET {JIRA_BASE_URL}/rest/api/3/search
  ?jql=project={PROJECT_KEY} AND statusCategory != Done AND issuetype in (Epic, Story) ORDER BY priority DESC, updated DESC
  &fields=summary,status,priority,assignee,labels,customfield_10014,customfield_10016,comment,updated,duedate
  &maxResults=50
```

**B. Issues updated in the last 14 days:**
```
GET {JIRA_BASE_URL}/rest/api/3/search
  ?jql=project={PROJECT_KEY} AND updated >= -14d ORDER BY updated DESC
  &fields=summary,status,priority,assignee,labels,updated,resolution
  &maxResults=30
```

**Headers:**
```
Authorization: Basic {base64(email:token)}
Accept: application/json
```

**Per issue, extract:**
- Key (e.g., OBS-42)
- Summary / title
- Status (To Do / In Progress / In Review / Done / Blocked)
- Priority (Highest / High / Medium / Low)
- Assignee name
- Epic link (parent)
- Story points (if available)
- Labels
- Due date (if set)
- Last updated date
- Latest comment text (truncated to 200 chars)

If Jira is unavailable or credentials are not provided: note this clearly and proceed with available sources.

---

### Step 3 — Fetch Data from Monday.com

Use `WebFetch` to call the Monday.com GraphQL API. Ask the user for their Monday.com credentials if not already provided.

**Required config (prompt user if missing):**
- `MONDAY_API_TOKEN` — API v2 personal token
- `MONDAY_BOARD_ID` — the board ID(s) containing feature work (ask user or try to discover)

**API endpoint:**
```
POST https://api.monday.com/v2
Headers:
  Authorization: {MONDAY_API_TOKEN}
  Content-Type: application/json
  API-Version: 2024-01
```

**GraphQL query:**
```graphql
{
  boards(ids: [{MONDAY_BOARD_ID}]) {
    name
    items_page(limit: 50) {
      items {
        id
        name
        state
        group { title }
        column_values {
          id
          title
          text
          value
        }
        updates(limit: 1) {
          body
          created_at
        }
        created_at
        updated_at
      }
    }
  }
}
```

**Per item, extract:**
- Item name
- Group/section (maps to feature area or sprint)
- Status column value
- Priority column value (if present)
- Owner/person column
- Timeline/due date column
- Latest update text (truncated to 200 chars)
- Last updated timestamp

If Monday.com is unavailable or credentials are not provided: note this clearly and proceed with available sources.

---

### Step 4 — Merge and Deduplicate

Combine data from all three sources (Jira, Monday.com, local file) into a single unified feature list.

**Deduplication rule:** If the same feature appears in multiple sources, merge them into one entry. Use this priority for field values: `Local file > Jira > Monday.com`.

**Unified feature record:**
```
Feature Name
├── Priority Tier: P0 / P1 / P2 / P3 (derive from source data — map to this scale)
├── Status: On Track / At Risk / Blocked / Shipped / Paused / Not Started
├── Source(s): [Jira | Monday | Local]
├── Owner: name
├── Due / Target Date: date or quarter
├── Key Progress Note: 1–2 sentence summary of latest update
├── Blockers: list (if any)
├── OKR Alignment: which OKR this maps to (from context files)
└── Risk Level: 🟢 Low / 🟡 Medium / 🔴 High
```

**Priority mapping logic:**
| Source Label | Unified Tier |
|---|---|
| Highest / Critical / P0 / Must Have | P0 |
| High / P1 / Should Have | P1 |
| Medium / P2 / Nice to Have | P2 |
| Low / P3 / Could Have | P3 |

**Status mapping logic:**
| Source Labels | Unified Status |
|---|---|
| In Progress, In Dev, Active, Working | On Track |
| Blocked, Impediment, On Hold, Stalled | Blocked |
| At Risk, Behind, Delayed | At Risk |
| Done, Shipped, Released, Closed | Shipped |
| To Do, Backlog, Not Started | Not Started |
| Paused, Deprioritized | Paused |

**Risk scoring (auto-assign):**
- 🔴 High: Status = Blocked OR past due date OR no owner assigned
- 🟡 Medium: Status = At Risk OR no due date set OR no recent update (>14 days old)
- 🟢 Low: Status = On Track AND has owner AND has due date

---

### Step 5 — Select Executive Highlights

From the merged list, select features for the executive summary using this logic:

**Always include:**
- All P0 features (regardless of status)
- All P1 features with status: On Track, At Risk, or Blocked
- Any feature with Risk Level 🔴 High

**Include if space allows (up to 6 total):**
- P1 features with status Not Started (if due date is within 60 days)
- P2 features with 🔴 High risk

**Exclude:**
- P3 features (unless they have a P0/P1 dependency)
- Paused features (omit entirely)
- Pure technical debt items with no user-facing impact
- Recently shipped features (do not include a "recent wins" section)

**For each selected feature, prepare:**
- P-tier badge + feature name + status pill
- One sentence: what it is + where it stands right now (include key metric vs target inline if available)
- Blocker/risk note only if status is At Risk or Blocked (one line)
- Owner + target quarter

---

### Step 6 — Generate the Executive Newsletter HTML

Produce a complete, self-contained HTML email newsletter. It must render perfectly when pasted into Gmail, Outlook, or any email client.

**Design Spec — Make Brand:**
- Font: system-ui, -apple-system, Arial, sans-serif
- Background: `#F8F5FF` (very light purple — Make brand wash)
- Header background: `#6C1DFF` (Make primary purple)
- Header text: `#FFFFFF`
- Accent / highlight: `#00C2A8` (Make teal)
- Section background: `#FFFFFF`
- Border / divider: `#E8DFFF` (soft purple)
- Body text: `#1A1A2E`
- Muted text: `#6B6B8A`
- P0 badge: background `#FF3B3B`, text `#FFFFFF`
- P1 badge: background `#6C1DFF`, text `#FFFFFF`
- P2 badge: background `#E8DFFF`, text `#6C1DFF`
- 🟢 On Track pill: `#E6F9F4` background, `#00A884` text
- 🟡 At Risk pill: `#FFF8E1` background, `#B8860B` text
- 🔴 Blocked pill: `#FFECEC` background, `#CC3333` text
- Max width: 680px, centered
- Inline styles only — no external CSS, no JavaScript
- Use `<table>` layout throughout for email client compatibility

**Sections — exactly in this order, no others:**

**1. Header**
- `make.` wordmark left, "Product Leadership Briefing" right-aligned — both in header purple bar
- Second row: "Observability · Top Features · Executive Summary" + date ("Week of [DATE]")
- No "Prepared by" line anywhere in the email

**2. At a Glance — 4 stat boxes in a single row**
- Features in Scope / 🟢 On Track / 🔴 At Risk or Blocked / ✅ Shipped This Month
- Each box: white card, 4px purple top border (`#6C1DFF`), large bold number, muted label below

**3. ⚠ Needs Attention** (only if any Blocked or At Risk features exist)
- One compact card per alert: left border color matches severity (red for Blocked, amber for At Risk)
- Inside each card: P-tier badge + feature name + status pill on one line; blocker description + italic ask on the next line
- Keep each alert to 2 lines of text maximum

**4. Top Priority Features**
- One card per feature, P0 first then P1, separated by 10px gap
- Card structure (left purple border, subtle shadow):
  - Row 1: P-tier badge + feature name (left) | status pill (right)
  - Row 2: One sentence — what it is + current progress + key metric vs target inline (e.g. "28% vs 40% target") — highlighted in the appropriate status color
  - Row 3 (only if At Risk or Blocked): risk/blocker note in red or amber
  - Row 4: 👤 Owner · 📅 Target quarter
- No section labels like "📌 What:" or "📊 Progress:" — just clean prose sentences

**5. Key Metrics — 2×2 grid**
- One card per OKR metric: metric name, large bold current value (colored by status), target below, progress bar, status label
- Colors: 🟢 `#00A884`, 🟡 `#B8860B`, 🔴 `#CC3333`
- Progress bar: `#E8E0FF` track, fill color matches status
- If live data unavailable: show OKR target values and note "Live data not yet connected"

**6. Footer — single line**
- `make.` wordmark + "Product Management — Observability · Next update: [DATE]"
- Dark background (`#1A1A2E`), no "Prepared by", no name attribution

---

### Step 7 — Present Draft to User

BEFORE sending anything, present the full HTML output to the user inline in the conversation.

Say:
> "Here's your executive newsletter draft. Please review and confirm before I send it. Let me know if you'd like to adjust any content, add features, change the tone, or update any details."

Show the rendered HTML (as a code block so they can inspect it) AND a plain-text summary of what's included:
- List of features included (with P-tier and status)
- Who it's addressed from
- Planned recipient(s)
- Send method (Gmail / Outlook / copy-paste instructions)

**Do NOT proceed to Step 8 until the user explicitly confirms with words like "send it", "looks good", "approved", "go ahead", or similar.**

If the user requests changes: apply them, show the updated draft, and ask for confirmation again.

---

### Step 8 — Send the Email

Once the user confirms, send the email.

**Ask for these details if not already known:**
- `FROM_NAME` — PM's full name (e.g., "Alex Rivera")
- `FROM_EMAIL` — PM's email address
- `TO_EMAIL` — recipient(s) — e.g., "CPO, CTO, CEO" or specific addresses
- `EMAIL_SUBJECT` — suggest a default: `"Make Observability — Feature Status Briefing · [Week of DATE]"`
- `SEND_METHOD` — Gmail (via Make scenario) or copy-paste to email client

**Option A — Make Scenario Blueprint (recommended for recurring use):**

Output a Make scenario blueprint the user can implement once and reuse:

```
Scenario: [Claude] Weekly Exec Feature Status Email

[1] Schedule Trigger (or Manual trigger)
    └─ Every: Friday
    └─ Time: 16:00 (user timezone)

[2] HTTP — POST Jira search API
    └─ URL: {JIRA_BASE_URL}/rest/api/3/search?jql=...
    └─ Headers: Authorization: Basic {base64(JIRA_EMAIL:JIRA_API_TOKEN)}

[3] HTTP — POST Monday.com GraphQL API
    └─ URL: https://api.monday.com/v2
    └─ Headers: Authorization: {MONDAY_API_TOKEN}

[4] HTTP — POST Claude API (claude-sonnet-4-6)
    └─ URL: https://api.anthropic.com/v1/messages
    └─ Headers: x-api-key: {ANTHROPIC_API_KEY}, anthropic-version: 2023-06-01
    └─ Body: {
         "model": "claude-sonnet-4-6",
         "max_tokens": 8192,
         "messages": [{
           "role": "user",
           "content": "You are the PM dashboard for Make Observability. Given the following Jira issues and Monday.com items, produce the full executive newsletter HTML following the Make brand newsletter format. Jira data: {{jira_response}}. Monday data: {{monday_response}}."
         }]
       }

[5] Text Parser — Extract HTML between <html> and </html>

[6] Gmail — Send Email
    └─ From: {FROM_NAME} <{FROM_EMAIL}>
    └─ To: {TO_EMAIL}
    └─ Subject: Make Observability — Feature Status Briefing · Week of {{formatDate(now; 'MMM D, YYYY')}}
    └─ Content Type: HTML
    └─ Body: {{extracted_html_from_step_5}}

[7] (Optional) Google Sheets — Log run
    └─ Append: Date | Features included | Blockers count | Sent to
```

**Option B — Direct send instructions (for one-off use):**

If the user wants to send immediately without setting up automation:
1. Copy the HTML from the code block above
2. Open Gmail → Compose → click the `<>` source button (or use a plugin like "Insert HTML")
3. Paste the HTML, set the subject line, add recipients, and send
4. Alternatively, forward the HTML via any email client that supports HTML paste

Deliver clear, copy-pasteable instructions based on their email client.

---

## Output Format

Deliver in this order:

### 1. Data Collection Summary
> "Fetched X items from Jira, Y items from Monday.com, and Z features from local file. [Any source errors or missing credentials noted here.]"

---

### 2. Merged Feature List (internal working table)

| Feature | Priority | Status | Risk | Owner | Target | Source(s) | OKR |
|---|---|---|---|---|---|---|---|
| Make Grid | P1 | On Track | 🟢 | ... | Q2 2026 | Jira + Monday | Grid adoption |

---

### 3. Executive Summary Draft (HTML)

```html
[Full self-contained HTML newsletter]
```

---

### 4. Confirmation Prompt

> "Here's your executive newsletter draft. X features included (Y on track, Z at risk/blocked, W shipped). Please review and let me know: **'send it'** to deliver, or tell me what to adjust."

---

### 5. (After confirmation) Send Confirmation + Make Scenario Blueprint

> "Email sent ✓ [or: Ready to send — copy the HTML above into Gmail]"
> Make scenario blueprint for automated recurring delivery.

---

## Rules (Always Apply)

- Never fabricate feature data — only use content retrieved from Jira, Monday.com, or the local file. If a source is unavailable, say so clearly.
- Always ask for credentials interactively if not provided — never hardcode or guess tokens.
- Do NOT send the email without explicit user confirmation — always show the draft first.
- The HTML must be fully self-contained with inline styles only — no external CSS, fonts, or scripts.
- P0 features must always be included in the executive summary if they exist.
- If no live data is available from any source, generate a template populated with the features from `context/feature-make-grid-observability.md` and mark each as "Status unknown — live data not connected".
- Include the Make scenario blueprint every time, even for one-off sends, so the user can set up automation.
- Always end with: "Next update: [date]. Enable the Make scenario to automate weekly delivery."
- Keep executive language: no jargon, no ticket numbers in the main body, outcomes-focused.
- **Format rules — never break these:**
  - No "Prepared by" or name attribution anywhere in the email
  - No "Recent Wins" or shipped features section
  - No roadmap timeline section
  - Feature cards use clean prose (no "📌 What:" / "📊 Progress:" labels)
  - Each feature card body is a maximum of 2 lines of text
  - Needs Attention alerts are a maximum of 2 lines each
  - Footer is a single line only
