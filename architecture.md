# Architecture & Data Flow

## Overview

This lab centres on a **Log Analytics workspace** with **Microsoft Sentinel** enabled.
Log sources stream in through data connectors; Sentinel runs analytics rules against that
data to raise incidents, and analysts run hunting queries proactively.

```
┌──────────────────────┐
│  Log sources         │
│  ------------------  │        data connectors
│  • Entra ID sign-ins │ ───────────────────────────┐
│  • Windows Security  │                             │
│    Events (via AMA)  │                             ▼
│  • Azure Activity    │              ┌────────────────────────────┐
└──────────────────────┘              │  Log Analytics Workspace   │
                                       │  Tables:                   │
                                       │   • SigninLogs             │
                                       │   • SecurityEvent          │
                                       │   • AzureActivity          │
                                       └─────────────┬──────────────┘
                                                     │
                                       ┌─────────────▼──────────────┐
                                       │     Microsoft Sentinel     │
                                       │  • Analytics rules ─► Incidents
                                       │  • Hunting queries         │
                                       │  • Watchlists              │
                                       │  • (optional) Playbooks    │
                                       └────────────────────────────┘
```

## Key tables used

| Table | Comes from | Used by |
|-------|-----------|---------|
| `SigninLogs` | Microsoft Entra ID connector | all `aad-*` detections |
| `SecurityEvent` | Windows Security Events (AMA) | `win-*` detections, hunting queries |
| `AzureActivity` | Azure Activity connector | (room for future detections) |

## Detection lifecycle in this lab

1. **Ingest** — connectors stream logs into the workspace.
2. **Detect** — a scheduled analytics rule (see `analytics-rules/`) runs KQL on a timer;
   matches create an **Incident**.
3. **Triage** — analyst reviews the incident, checks entities (IP, account), decides
   true/false positive.
4. **Hunt** — separately, hunting queries (`hunting/`) look for activity that no rule
   caught yet.
5. **Tune** — thresholds and allow-lists are adjusted to cut false positives.

## Notes

- Windows process-command-line hunting (`encoded-powershell.kql`) needs **command-line
  auditing** enabled (Event ID 4688 with CommandLine) on the endpoints.
- Country-based detection depends on `LocationDetails` being populated in `SigninLogs`.
