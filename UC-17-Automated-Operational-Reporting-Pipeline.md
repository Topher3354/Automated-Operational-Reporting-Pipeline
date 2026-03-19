# Use Case 17 — Automated Operational Reporting Pipeline

**Category:** Data Pipelines & Reporting
**Author:** Christopher Ayodeji Ojo — [Trublshut](https://whop.com/trublshut)
**Tools:** Python · n8n · Power BI REST API · SharePoint · Email (Graph API)

---

## 🔴 The Problem

Operations managers wait for weekly or monthly reports that are manually compiled by analysts — pulling data from multiple systems, copying into spreadsheets, formatting, and emailing. Reports arrive late, stale, and inconsistently formatted, undermining decision-making.

The analyst spends most of their time doing mechanical data work — not analysis. And by the time the report lands in the manager's inbox, the data is already out of date.

---

## ✅ The Solution

A fully automated reporting pipeline that collects data from all operational sources on a schedule, processes and structures it using Python, pushes it to Power BI, and delivers a formatted report to stakeholders automatically — always on time, always consistent, always current.

---

## ⚙️ How It Works

```
n8n scheduled trigger (daily 7am / weekly Monday 6am)
        ↓
Parallel data pulls:
→ Jira API: ticket volumes, resolution times, SLA rates
→ SharePoint: internal operational records
→ CRM API: pipeline, deal activity, conversion rates
→ Internal APIs: system-specific metrics
        ↓
Python: clean and normalise all data
(standardise field names, date formats, handle nulls)
        ↓
Python: calculate KPIs
→ SLA compliance rate (% tickets resolved within SLA)
→ Average resolution time by category
→ Ticket volume trends (week-on-week, month-on-month)
→ Open ticket age distribution
→ Team workload distribution
        ↓
Push processed dataset to Power BI via REST API
(trigger dataset refresh)
        ↓
Power BI dashboard updates automatically
        ↓
Export PDF snapshot of key report pages
        ↓
Email stakeholders: PDF attached + key metrics summary in email body
        ↓
Archive report to SharePoint (timestamped filename)
        ↓
Log pipeline run: data sources pulled, KPIs calculated, send status
```

---

## 🛠️ Tech Stack

| Tool | Role |
|------|------|
| n8n | Scheduling + orchestration + API connections |
| Python (pandas) | Data processing + KPI calculation |
| Power BI REST API | Dataset refresh + PDF export |
| SharePoint | Data source + report archive |
| Email (Graph API / SMTP) | Stakeholder report delivery |

---

## 🔧 Key Logic

```python
import pandas as pd
from datetime import datetime, timedelta

# Load raw data from API responses passed by n8n
tickets_df = pd.DataFrame(jira_response['issues'])

# Calculate SLA compliance
tickets_df['resolved_within_sla'] = (
    pd.to_datetime(tickets_df['resolved_at']) <=
    pd.to_datetime(tickets_df['sla_due'])
)
sla_rate = tickets_df['resolved_within_sla'].mean() * 100

# Calculate average resolution time by category
tickets_df['resolution_hours'] = (
    pd.to_datetime(tickets_df['resolved_at']) -
    pd.to_datetime(tickets_df['created_at'])
).dt.total_seconds() / 3600

avg_resolution = tickets_df.groupby('category')['resolution_hours'].mean()

# Week-on-week volume trend
this_week = tickets_df[tickets_df['created_at'] >= (datetime.now() - timedelta(days=7))]
last_week = tickets_df[(tickets_df['created_at'] >= (datetime.now() - timedelta(days=14))) &
                       (tickets_df['created_at'] < (datetime.now() - timedelta(days=7)))]
volume_change_pct = ((len(this_week) - len(last_week)) / len(last_week)) * 100
```

- **Stakeholder segmentation:** different stakeholder groups receive different sections — IT managers get technical metrics, executive team gets summary KPIs only
- **Anomaly flagging:** Python checks if any KPI falls outside historical bounds (e.g. SLA rate drops >10% from 4-week average) — anomalies highlighted in red in the email summary

---

## 📊 Outcomes

- ✅ Reports delivered automatically on schedule — zero manual effort
- ✅ Stakeholders always working with current data, not week-old snapshots
- ✅ Consistent format and calculation methodology every report cycle
- ✅ Analyst time freed for insight and strategy — not data compilation
- ✅ Anomalies flagged automatically — no one has to spot them manually

---

## 📁 How to Use This

1. In n8n, create a Schedule trigger (Cron: `0 7 * * 1` for weekly Monday 7am)
2. Add HTTP Request nodes for each data source — Jira, CRM, SharePoint, internal APIs
3. Pass combined raw data to a Python Code node — run cleaning and KPI calculations
4. Use n8n's HTTP Request node to call the Power BI REST API: `POST /datasets/{id}/refreshes`
5. Poll for refresh completion, then call the Power BI Export API to download PDF
6. Send stakeholder email via Microsoft Graph API with PDF attached and key metrics in the body
7. Upload the PDF to SharePoint with a timestamped filename
8. Write pipeline run metadata to a SharePoint log list

---

## 🔗 Related Use Cases

- [Use Case 11 — Finance Reconciliation Data Pipeline](./UC-11-Finance-Reconciliation-Data-Pipeline.md)
- [Use Case 18 — Multi-Source Data Aggregation & Transformation](./UC-18-Multi-Source-Data-Aggregation.md)

---

*© 2026 Christopher Ayodeji Ojo · Trublshut AI Automation · christopherayodeji131@gmail.com*