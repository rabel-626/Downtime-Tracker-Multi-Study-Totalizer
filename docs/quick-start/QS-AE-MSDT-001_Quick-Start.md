---
documentId: QS-AE-MSDT-001
title: Multi-Study Downtime Totalizer Quick Start
tool: Downtime-Tracker-Multi-Study-Totalizer
documentVersion: 1.1
toolVersion: 1.2.0
status: Review Draft
updated: 2026-09-09
---

# Multi-Study Downtime Totalizer — Quick Start

## Purpose

The **Abel Engineering Multi-Study Downtime Totalizer** combines multiple Downtime Tracker studies into one repeatable downtime analysis. It is intended for recurring-loss identification, cross-study Pareto analysis, comparison by location or shift, study-level drilldown, production-impact estimation, and controlled grouping of related downtime tags.

The current workflow is repository-centered:

```text
02_DOWNTIME
    ↓
LOAD DOWNTIME STUDIES
    ↓
Filter / Group / Analyze
    ↓
SAVE ANALYSIS
    ↓
04_MULTI_STUDY_DOWNTIME
```

> **Important:** The Totalizer does not modify the source Downtime Tracker studies. It reads them into the analysis and saves a separate Multi-Study analysis record.

---

## What You Need Before Starting

For the normal workflow, you need:

- A supported desktop browser, preferably **Microsoft Edge** or **Google Chrome**.
- Access to the Abel Engineering **Study Repository**.
- One or more completed Downtime Tracker studies under `02_DOWNTIME`.
- A clear analytical question, such as:
  - Which losses dominate this line across the last month?
  - Are Red-shift losses materially different from Black-shift losses?
  - Which recurring tags should be consolidated into a common loss family?
  - Which loss categories account for the largest estimated production impact?

For older studies outside the repository, use **Legacy File Load**.

---

## Quick Workflow

### 1. Connect the Study Repository

Select **REPOSITORY CONNECTION / RECONNECT** and choose the approved repository root:

```text
Engineering Study Hub - Study Repository
```

or:

```text
Study Repository
```

The application expects the standard repository routes, including:

```text
02_DOWNTIME
04_MULTI_STUDY_DOWNTIME
```

Once permission is granted, the browser can remember the repository handle. Browser permission may still need to be re-granted in a later session.

---

### 2. Identify the Analysis

Complete the **Multi-Study Analysis Identity** fields:

| Field | Recommended use |
|---|---|
| Analysis Name / Identifier | A clear human-readable analysis name |
| Organizer | Engineer, team, or department responsible for the analysis |
| Analysis Date | Date the analysis is being created or revised |
| Description | Scope, question, study population, or comparison being performed |

Example:

```text
Analysis Name: Slitter downtime baseline — Q3 2026
Organizer: Industrial Engineering
Description: Compare all normal-production downtime studies from Lines 12–16 for Q3.
```

> **Required for repository save:** An Analysis Name / Identifier must be entered before **SAVE ANALYSIS** can complete.

---

### 3. Load Downtime Studies

Select **LOAD DOWNTIME STUDIES**.

The repository browser scans `02_DOWNTIME` for compatible Downtime Tracker study JSON files and lets you choose multiple source studies.

Before continuing, confirm:

- the expected number of studies loaded;
- the correct locations and lines are represented;
- study dates and shifts match your intended scope;
- no study has been loaded twice.

> **Important:** The Totalizer does not currently deduplicate loaded studies. A repeated source study will be counted again.

---

### 4. Understand the Source Study Scope

The current Totalizer preserves the Downtime Tracker's **source inclusion logic**.

When a current source study provides an authoritative study-scope summary, the Totalizer uses it. Otherwise, it reconstructs source scope from the source study's reason-code definitions and `Include in Summary` settings.

At the source-study level:

```text
Included Study Time
= Total Logged Study Time − Source-Excluded Event Time
```

```text
Included Downtime
= Σ source-included downtime-event duration
```

```text
Included Uptime
= MAX(0, Included Study Time − Included Downtime)
```

This matters because the Totalizer's active reason-code filters are an **additional analysis layer**. They do not rewrite the original source study.

---

### 5. Set the Study Slicer

Use the study controls to define the active Pareto view.

You can filter by:

- **Location**
- **Shift**
- **Study sort order**
- **All matching studies** or one individual study

The main Pareto, headline metrics, and production-impact view recalculate from the visible study set.

Use the individual-study selector when you want to inspect one source study without unloading the others.

---

### 6. Review Reason-Code Inclusion

Every source-included reason-code tag is available to the analysis. Use the reason-code checkboxes to decide which tags belong in the **current analytical view**.

Typical reasons to remove a code from the current Pareto include:

- a planned condition you do not want in the current loss analysis;
- a non-loss observation category;
- a code outside the question being studied;
- a tag intentionally excluded for a particular comparison.

This is an analysis filter only. The source Downtime Tracker file remains unchanged.

---

### 7. Group Similar Tags When Appropriate

Use grouping only when different source tags represent the same analytical loss family.

Example:

```text
Group: Material Feed
  ├─ FEEDER_JAM
  ├─ MAG_EMPTY
  └─ FEED_MISALIGN
```

Two group modes are supported:

| Mode | Effect |
|---|---|
| **Show this group combined** | Member-code events are rolled into one Pareto contributor named after the group. |
| **Show this group as original tags** | The group organizes the tags, but each original tag remains separate in the Pareto. |

A source tag can belong to only one group at a time.

> **Good practice:** Group by a documented analytical rationale, not simply because two labels sound similar.

---

### 8. Review the Pareto

The Combined Downtime Pareto ranks the active contributors by total downtime.

For each Pareto row:

```text
Percent of Downtime
= Contributor Downtime / Total Included Downtime in Current View
```

```text
Cumulative Percent
= Running Sum of Ranked Contributor Downtime
  / Total Included Downtime in Current View
```

The table provides:

- rank;
- effective reason code or group name;
- description;
- event count;
- downtime;
- downtime minutes;
- percent of current-view downtime;
- cumulative percent;
- studies affected.

Expand a Pareto row to review the underlying source events.

---

### 9. Review Production Impact

For each source study, the base production model is:

```text
Potential Production
= Production Speed × Included Study Hours
```

```text
Estimated Lost Production
= Production Speed × Included Downtime Hours
```

```text
Estimated Available Production
= Production Speed × Included Uptime Hours
```

For the **current filtered Pareto view**, potential production remains based on the visible studies' full included study scope, while estimated lost production is recalculated from the reason codes currently included in the analysis.

Therefore:

```text
Filtered Availability
= (Potential Production − Filtered Estimated Loss)
  / Potential Production
```

> **Interpretation:** Filtering out a reason code removes that code from the current loss view; it does not shorten the source study's observed production opportunity.

---

### 10. Check the Data Calculation Tab

The **Data Calculation** table is intentionally different from the active Pareto view.

It uses:

- **all loaded studies**;
- each study's complete source-included downtime scope;
- source production speed;
- source included study time.

It does **not** mirror every current location/shift/study/tag filter or grouping decision.

Use it as the normalized loaded-study calculation reference, not as a duplicate of the filtered Pareto.

---

### 11. Check Loaded Study Reference

Before finalizing, expand the source studies and inspect:

- source scope basis;
- total logged time;
- source-excluded event time;
- included study time;
- included downtime;
- included uptime;
- timer segments;
- included event records;
- source-excluded event audit records;
- production speed.

This is the best place to diagnose unexpected totals.

---

### 12. Save the Analysis

Select **SAVE ANALYSIS**.

A repository analysis record is assigned an ID similar to:

```text
MDA-20260909-A1B2
```

The normal folder route is:

```text
04_MULTI_STUDY_DOWNTIME/
└── LOCATION or MULTI_LOCATION/
    └── LINE or MULTI_LINE/
        └── YYYY/
            └── YYYY-MM/
                └── MDA-RECORD-ID/
```

The save package contains:

```text
MDA-...__ANALYSIS.json
MDA-...__REPORT.xls
MDA-...__MANIFEST.json
```

The Analysis JSON stores the source-study references and the current analysis configuration so **OPEN SAVED ANALYSIS** can restore the analysis later.

---

## Final Review Checklist

Before using the result as an engineering baseline, confirm:

- [ ] The intended source studies are loaded once each.
- [ ] Location and shift filters match the question being analyzed.
- [ ] Source study scope looks reasonable.
- [ ] Production speed is available where production-loss results are being used.
- [ ] Reason-code inclusion matches the current analytical intent.
- [ ] Grouped tags genuinely represent the same analytical loss family.
- [ ] Identical abbreviations from different categories have been reviewed.
- [ ] The Pareto and Data Calculation scopes are not being confused.
- [ ] Analysis identity fields are complete.
- [ ] The analysis has been saved to `04_MULTI_STUDY_DOWNTIME` if it must be reopened or audited later.

---

## Related Documents

- **WI-AE-MSDT-001** — Multi-Study Downtime Analysis Procedure
- **REF-AE-MSDT-001** — Downtime Aggregation & Pareto Methodology
- **LIM-AE-MSDT-001** — Limitations
- **DATA-AE-MSDT-001** — Data Handling & Repository Records
