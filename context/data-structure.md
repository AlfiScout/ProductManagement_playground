# Make Platform — Dummy Data Structure Reference

> ⚠️ This is a DUMMY / ILLUSTRATIVE file for PM context only.
> Data structures, field names, and values are representative approximations — not real production schemas.
> Use this to reason about data availability, event design, and metric feasibility.

---

## Core Entities

### Organization
```json
{
  "org_id": "org_abc123",
  "name": "Acme Corp",
  "plan": "teams",                  // free | core | pro | teams | enterprise
  "operation_limit_monthly": 100000,
  "operations_used_current_cycle": 43200,
  "created_at": "2022-11-15T09:00:00Z",
  "region": "eu-west-1",
  "sso_enabled": true,
  "team_count": 4
}
```

### Team
```json
{
  "team_id": "team_xyz789",
  "org_id": "org_abc123",
  "name": "Marketing Ops",
  "operation_limit": 20000,         // team-level credit cap
  "operations_used": 8100,
  "member_count": 6,
  "scenario_count": 23
}
```

### User
```json
{
  "user_id": "usr_001",
  "org_id": "org_abc123",
  "team_id": "team_xyz789",
  "email": "alice@acme.com",
  "role": "admin",                  // admin | member | viewer
  "last_active_at": "2025-03-20T14:32:00Z",
  "created_at": "2023-01-10T08:00:00Z"
}
```

---

## Scenario (Workflow)

```json
{
  "scenario_id": "scen_001",
  "org_id": "org_abc123",
  "team_id": "team_xyz789",
  "created_by": "usr_001",
  "name": "New Lead → CRM + Slack Notify",
  "status": "active",              // active | inactive | error | draft
  "trigger_type": "webhook",       // webhook | schedule | instant | manual
  "schedule_cron": null,
  "last_run_at": "2025-03-23T18:45:00Z",
  "last_run_status": "success",    // success | error | warning | incomplete
  "run_count_total": 14203,
  "run_count_30d": 1840,
  "error_rate_7d": 0.02,           // 2% of runs errored in last 7 days
  "avg_duration_ms": 1240,
  "operations_per_run_avg": 4.2,
  "module_count": 7,
  "is_active": true,
  "created_at": "2023-06-01T10:00:00Z",
  "updated_at": "2025-03-10T16:00:00Z",
  "tags": ["lead-gen", "crm", "notifications"]
}
```

---

## Module (Step within a Scenario)

```json
{
  "module_id": "mod_a1",
  "scenario_id": "scen_001",
  "position": 3,
  "app": "slack",
  "action": "send_message",
  "connection_id": "conn_slack_01",
  "is_router": false,
  "has_error_handler": true,
  "config": {
    "channel": "#sales-alerts",
    "message_template": "New lead: {{bundle.name}}"
  }
}
```

---

## Connection (App Authentication)

```json
{
  "connection_id": "conn_slack_01",
  "org_id": "org_abc123",
  "app": "slack",
  "display_name": "Acme Slack Workspace",
  "status": "active",              // active | expired | revoked | error
  "created_by": "usr_001",
  "created_at": "2023-05-15T00:00:00Z",
  "last_verified_at": "2025-03-22T00:00:00Z",
  "used_by_scenario_count": 8
}
```

---

## Scenario Run (Execution)

```json
{
  "run_id": "run_20250323_001",
  "scenario_id": "scen_001",
  "org_id": "org_abc123",
  "team_id": "team_xyz789",
  "triggered_by": "webhook",
  "status": "success",             // success | error | warning | incomplete | timeout
  "started_at": "2025-03-23T18:45:00Z",
  "finished_at": "2025-03-23T18:45:01Z",
  "duration_ms": 1102,
  "operations_consumed": 4,
  "bundle_count": 1,
  "custom_run_name": null,         // user-defined label for tracking
  "error_code": null,
  "error_message": null,
  "error_module_id": null,
  "is_replay": false,
  "replayed_from_run_id": null
}
```

---

## Module Execution (Step-level log)

```json
{
  "module_execution_id": "mexec_001",
  "run_id": "run_20250323_001",
  "module_id": "mod_a1",
  "scenario_id": "scen_001",
  "position": 3,
  "app": "slack",
  "action": "send_message",
  "status": "success",
  "started_at": "2025-03-23T18:45:00.800Z",
  "finished_at": "2025-03-23T18:45:00.950Z",
  "duration_ms": 150,
  "input_bundle": {
    "name": "John Doe",
    "email": "john@example.com"
  },
  "output_bundle": {
    "message_id": "slack_msg_xyz",
    "timestamp": "1711215900.123456"
  },
  "error": null,
  "operations_consumed": 1
}
```

---

## Observability Events (Instrumentation Layer)

> These are events Make (ideally) emits to support observability features.
> Availability status: A = available, P = partially available, M = missing

### Scenario-level events
| Event | Availability | Notes |
|---|---|---|
| `scenario.run.started` | A | Fired on every execution start |
| `scenario.run.completed` | A | Fired on success/failure |
| `scenario.run.errored` | A | Fired when run ends in error |
| `scenario.status.changed` | P | Fired on active/inactive toggle; not on auto-disable |
| `scenario.updated` | P | Config change, no field-level diff |
| `scenario.deleted` | A | Fired on deletion |

### Module-level events
| Event | Availability | Notes |
|---|---|---|
| `module.executed` | A | Per-step execution event |
| `module.errored` | A | Includes error code and message |
| `module.skipped` | M | Not currently emitted for router branches not taken |

### Connection events
| Event | Availability | Notes |
|---|---|---|
| `connection.expired` | P | Fired, but with ~1hr delay |
| `connection.revoked` | A | Immediate |
| `connection.verified` | M | Not tracked |

### Consumption events
| Event | Availability | Notes |
|---|---|---|
| `operations.consumed` | A | Per-run total |
| `operations.limit.warning` | P | Fires at 80%, not configurable |
| `operations.limit.reached` | A | Fires at 100% |

---

## Make Grid Data Model (Conceptual)

```json
{
  "grid_snapshot_id": "grid_snap_001",
  "org_id": "org_abc123",
  "generated_at": "2025-03-23T18:00:00Z",
  "nodes": [
    {
      "node_id": "scen_001",
      "type": "scenario",
      "label": "New Lead → CRM + Slack",
      "status": "active",
      "error_rate_7d": 0.02,
      "operations_30d": 7728
    },
    {
      "node_id": "conn_slack_01",
      "type": "connection",
      "label": "Acme Slack Workspace",
      "app": "slack",
      "status": "active"
    }
  ],
  "edges": [
    {
      "from": "scen_001",
      "to": "conn_slack_01",
      "type": "uses_connection"
    }
  ]
}
```

---

## Data Retention (Illustrative)
| Data Type | Retention Period | Notes |
|---|---|---|
| Scenario run records | 30 days (Free/Core), 90 days (Pro+) | Configurable on Enterprise |
| Module execution logs | 30 days | Full input/output payload |
| Error logs | 90 days | All plans |
| Analytics aggregates | 12 months | Rolled-up, not raw |
| Audit trail | 12 months (Enterprise) | Not available on lower tiers |

---

## Known Data Gaps (for PM reference)
1. No user-level attribution on scenario runs (who triggered it, not just which scenario)
2. No field-level change tracking on scenario config updates
3. Module `skipped` event not emitted — can't track router branch distribution
4. Connection health checks not event-driven — inferred from run errors only
5. No cross-org / MSP-level rollup for agencies managing multiple client orgs
