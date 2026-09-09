---
documentId: WI-AE-MSDT-001
title: Multi-Study Downtime Analysis Procedure
tool: Downtime-Tracker-Multi-Study-Totalizer
documentVersion: 1.1
toolVersion: 1.2.0
status: Review Draft
updated: 2026-09-09
---

# Multi-Study Downtime Totalizer — Analysis Procedure

## 1. Purpose

This work instruction defines the recommended procedure for creating, reviewing, saving, reopening, and exporting a multi-study downtime analysis using the **Abel Engineering Multi-Study Downtime Totalizer**.

The procedure is intended to preserve source-study traceability while allowing an analyst to create an independent analysis layer consisting of:

- source-study selection;
- location and shift filtering;
- individual-study slicing;
- reason-code inclusion/exclusion;
- related-tag grouping;
- Pareto analysis;
- production-impact estimation;
- reusable repository analysis records.

---

## 2. Scope

Use this procedure when analyzing two or more Downtime Tracker studies, or when using the Totalizer to perform a controlled filtered/grouped review of a single repository study.

Typical applications include:

- monthly or quarterly downtime Pareto reviews;
- cross-shift loss comparison;
- cross-line recurring-loss review;
- before/after improvement comparison;
- repeated-study consolidation;
- production-loss estimation across a selected source population.

This tool is an **analysis utility**, not the authoritative source of the original downtime observations. The source-of-record remains the Downtime Tracker study package.

---

## 3. Required Inputs

### 3.1 Preferred source

Current repository-based Downtime Tracker study records under:

```text
02_DOWNTIME
```

The repository loader scans for study JSON files whose file type is compatible with the Downtime Tracker source contract.

### 3.2 Legacy source

Use **LEGACY FILE LOAD** only when required for older local JSON studies that are not available under `02_DOWNTIME`.

Legacy files must:

- use the `.json` extension;
- parse as valid JSON;
- contain an `events` array.

### 3.3 Browser

Use current desktop Edge or Chrome for the full Study Repository workflow because repository access depends on browser folder-access capabilities.

---

## 4. Definitions

| Term | Definition |
|---|---|
| **Source Study** | One Downtime Tracker study loaded into the Totalizer. |
| **Source-Included Event** | An event included by the source study's own `Include in Summary` logic. |
| **Source-Excluded Event** | An event excluded by the source study and preserved for audit but not included in the Totalizer's source downtime totals. |
| **Visible Study** | A loaded study that remains after current location, shift, and individual-study filtering. |
| **Analysis-Included Code** | A source-included reason code currently selected for the active Pareto. |
| **Effective Code** | The displayed Pareto key after optional tag grouping. |
| **Combined Group** | A tag group that rolls multiple source codes into one Pareto contributor. |
| **Separate Group** | A tag group used for organization while preserving original Pareto contributors. |
| **Included Study Time** | Source study time after source-excluded event time is removed from the analytical denominator. |
| **Analysis Record** | Reusable Multi-Study configuration saved under `04_MULTI_STUDY_DOWNTIME`. |

---

## 5. Procedure

### 5.1 Open the Totalizer

1. Open the published Totalizer in Edge or Chrome.
2. Confirm the application loads without browser errors.
3. If the repository badge is not connected, select **REPOSITORY CONNECTION / RECONNECT**.
4. Select the approved Study Repository root.
5. Grant read/write access when prompted.

The application accepts repository roots named:

```text
Engineering Study Hub - Study Repository
```

or:

```text
Study Repository
```

---

### 5.2 Define the Analysis Identity

Complete:

1. **Analysis Name / Identifier**
2. **Organizer**
3. **Analysis Date**
4. **Description**

The Analysis Name should tell another reviewer what the analysis represents without opening the record.

Recommended pattern:

```text
[Asset / Area] — [Question or Scope] — [Period]
```

Example:

```text
Slitter 12–16 — Recurring downtime baseline — Q3 2026
```

The description should record analytical boundaries, such as excluded shifts, product families, commissioning periods, or other deliberate scope choices.

---

### 5.3 Load Source Studies from the Repository

1. Select **LOAD DOWNTIME STUDIES**.
2. Allow the application to scan `02_DOWNTIME`.
3. Search or review the repository list.
4. Select the studies required for the analysis.
5. Select **Load Selected Studies**.
6. Confirm the status message reports the expected number of studies.

The repository source browser uses source metadata such as:

- location;
- line;
- date;
- shift;
- product;
- source record ID.

#### Required verification

After loading, review the **Loaded Study Reference** and confirm each intended source appears once.

> **Warning:** The Totalizer does not prevent the same source study from being loaded more than once. Duplicate studies duplicate their downtime, observed time, event counts, and production impact.

---

### 5.4 Load Legacy Sources When Necessary

1. Select **LEGACY FILE LOAD**.
2. Choose individual JSON files or a legacy study folder.
3. Confirm only the intended studies loaded.
4. Record in the Analysis Description that legacy sources were used.

Legacy loading is a source-ingestion path only. It does not become an alternate save destination. **SAVE ANALYSIS** still writes to `04_MULTI_STUDY_DOWNTIME`.

> **Traceability note:** Repository-loaded sources carry repository record/path references. Legacy-loaded sources generally rely more heavily on filenames for later identification.

---

### 5.5 Validate Source Scope

For each study, expand **Loaded Study Reference** and review the source scope line.

The current application distinguishes:

```text
Total Logged Study Time
Source-Excluded Event Time
Included Study Time
Included Downtime
Included Uptime
```

Current source-scope precedence is:

1. use an authoritative source study-scope summary when present;
2. otherwise reconstruct the scope from source Downtime Tracker data.

When reconstruction is required:

```text
Included Study Time
= MAX(0, Total Logged Study Time − Source-Excluded Event Time)
```

```text
Included Downtime
= Σ source-included completed-event duration
```

```text
Included Uptime
= MAX(0, Included Study Time − Included Downtime)
```

Resolve unexpected source scope before proceeding.

---

### 5.6 Verify Observed-Time Basis

If a source study does not provide the current authoritative scope summary, total logged time is derived by this precedence:

1. positive manual study duration;
2. sum of valid completed study segments;
3. elapsed span between earliest and latest valid event timestamps;
4. zero if no usable basis exists.

The Loaded Study Reference identifies the source used.

> **Review trigger:** An event-timestamp span is a fallback, not proof that the entire span was actively observed. Investigate any study using this fallback before treating the denominator as an engineering baseline.

---

### 5.7 Apply Study Filters

Use the study slicer to select:

- location;
- shift;
- sort order;
- all matching studies or one selected study.

These controls change the active Pareto population without removing the other loaded studies from memory.

Recommended use:

| Question | Suggested filter |
|---|---|
| What are the total losses across the full loaded set? | All locations / All shifts / All matching studies |
| Which losses dominate one shift? | Select the shift, leave study = All matching |
| Isolate one study for troubleshooting | Select the individual study |
| Compare one location at a time | Select location, keep study = All matching |

---

### 5.8 Review Reason-Code Inclusion

Reason-code inclusion is an **analysis-layer filter** applied after source-study inclusion.

1. Review every reason-code tag available in the active study population.
2. Leave codes checked when they belong in the current Pareto.
3. Uncheck codes intentionally excluded from the current question.
4. Record any non-obvious exclusion in the Analysis Description or external review notes.

The current Pareto includes an event only when:

```text
Source Study Includes Event
AND
Current Analysis Includes Event Code
AND
Study Is Visible in Current Slicer
```

---

### 5.9 Review Code Identity

The Totalizer aggregates Pareto contributors primarily by normalized **reason-code abbreviation**.

Source category information is used when reconstructing source inclusion, but category is not part of the active Pareto aggregation key.

Therefore, if two source studies use:

```text
Category A / Code JAM
Category B / Code JAM
```

the Totalizer will aggregate both under:

```text
JAM
```

unless the source coding standard uses distinct abbreviations.

> **Required review:** Before finalizing a multi-line or multi-template analysis, confirm identical abbreviations mean the same operational loss across the source population.

---

### 5.10 Create Analytical Groups

Use groups only when a broader analytical roll-up is useful and defensible.

#### To create a group

1. Enter a group name.
2. Select **Add Grouping Row**.
3. Drag tags into the group or add currently checked ungrouped tags to the last group.
4. Choose the group display mode.

#### Combined mode

```text
Source codes A + B + C
        ↓
Pareto contributor = Group Name
```

#### Separate mode

```text
Source codes A + B + C
        ↓
Grouped for organization only
        ↓
Pareto still displays A, B, C separately
```

One source code can belong to only one group.

#### Group-control behavior

- group visibility can include or exclude all member tags;
- individual grouped tags can still be included/excluded independently;
- renaming a combined group changes the Pareto label, not the source events;
- removing a group restores original-code analysis behavior.

---

### 5.11 Review Headline Metrics

The active analysis shows:

- loaded study count;
- active event count;
- active downtime;
- visible included study time;
- downtime percentage.

For the active view:

```text
Downtime %
= Active Included Downtime / Visible Included Study Time
```

Do not interpret this as OEE. It is a downtime/availability measure within the defined included source scope.

---

### 5.12 Review the Pareto

The application:

1. resolves each analysis-included source event to an effective code;
2. sums duration by effective code;
3. sorts contributors from largest to smallest downtime;
4. calculates percent of active downtime;
5. calculates cumulative percent;
6. counts distinct source studies affected.

For contributor `i`:

```text
Pareto Share_i
= Downtime_i / Σ Active Pareto Downtime
```

```text
Cumulative Share_i
= Σ Downtime_1..i / Σ Active Pareto Downtime
```

Expand major contributors and inspect event-level detail before assigning improvement ownership solely from the aggregate label.

---

### 5.13 Review Production Metrics

For each source study:

```text
Potential Production
= Production Speed × Included Study Time / 3600
```

```text
Source Estimated Loss
= Production Speed × Source Included Downtime / 3600
```

```text
Source Estimated Available Production
= Production Speed × Included Uptime / 3600
```

For the current filtered Pareto view:

```text
Filtered Loss
= Production Speed × Analysis-Included Downtime / 3600
```

Potential production is not reduced merely because a reason code is filtered out.

This provides a useful sensitivity view of the selected loss set, but it assumes the entered production speed is an appropriate linear production reference.

---

### 5.14 Review Data Calculation

Use the Data Calculation table to review the normalized source-study model.

It uses **all loaded studies**, regardless of the current Pareto study slicer or reason-code grouping.

The total row is calculated by summing study-level:

- included study seconds;
- source included downtime seconds;
- potential units;
- lost units;
- available units.

Combined downtime percentage is:

```text
Combined Downtime %
= Σ Source Included Downtime
  / Σ Included Study Time
```

Do not compare a filtered Pareto percentage to the Data Calculation total without first confirming the scopes are equivalent.

---

### 5.15 Review Loaded Study Reference

Expand any study requiring audit.

Review:

- source scope summary;
- segment timestamps and durations;
- included event timing records;
- source-excluded events;
- notes;
- production speed;
- source repository identity when applicable.

This table should be used whenever a total seems inconsistent with the originating Tracker study.

---

### 5.16 Save the Analysis

Before saving:

- confirm at least one study is loaded;
- confirm the Analysis Name is populated;
- confirm the intended filters and tag groups are active;
- verify the source-study list.

Select **SAVE ANALYSIS**.

The Totalizer assigns or reuses a repository record ID:

```text
MDA-YYYYMMDD-XXXX
```

The destination is:

```text
04_MULTI_STUDY_DOWNTIME/
└── [single location or MULTI_LOCATION]/
    └── [single line or MULTI_LINE]/
        └── YYYY/
            └── YYYY-MM/
                └── MDA-RECORD-ID/
```

The application writes:

```text
MDA-...__ANALYSIS.json
MDA-...__REPORT.xls
MDA-...__MANIFEST.json
```

The Analysis JSON contains:

- analysis identity;
- source-study references;
- calculation contract;
- current excluded reason codes;
- selected study/location/shift;
- sort mode;
- tag groups and group modes;
- summary data.

---

### 5.17 Reopen a Saved Analysis

Select **OPEN SAVED ANALYSIS**.

The application scans `04_MULTI_STUDY_DOWNTIME` for compatible analysis JSON files.

When an analysis is selected, it attempts to resolve each source study from `02_DOWNTIME` using, in order of available identity:

1. repository record ID;
2. repository relative path;
3. repository file path;
4. filename.

If some sources are missing but at least one resolves, the analysis can open with a warning and the missing sources are not loaded.

If none of the required sources can be resolved, the analysis cannot be reconstructed from the repository.

After opening, verify:

- source count;
- missing-source warning status;
- filters;
- group mappings;
- headline totals.

---

### 5.18 Export Reports

#### Excel-compatible report

The workbook-style `.xls` export includes multiple analytical scopes:

- Summary
- Pareto Breakdown
- Study Reference
- Data Calculation
- Event Timing Records
- Source-Excluded Event Audit
- Study Segment Timing
- Tag Group Mapping

Review the scope of each table before interpreting cross-sheet differences.

#### Pareto PDF

The print view is optimized for the current Pareto and visible-study context.

For best results:

- use landscape orientation;
- enable **Background graphics**;
- review the table expansion state before printing.

---

## 6. Data Quality Requirements

A multi-study result should not be treated as a baseline until the analyst has reviewed:

- source study completeness;
- duplicate source risk;
- source-inclusion settings;
- reason-code semantic consistency;
- category/code collision risk;
- study-time basis;
- production-speed completeness;
- filter scope;
- grouping rationale;
- missing-source warnings on reopened analyses.

---

## 7. Common Errors and Corrective Actions

| Condition | Likely cause | Corrective action |
|---|---|---|
| Downtime too high | Source loaded twice | Remove duplicate study and recalculate |
| Pareto differs from Tracker | Additional Totalizer code filter/grouping, or different visible-study scope | Align filters and compare source scope |
| Two different causes merged | Same reason-code abbreviation used across categories | Correct coding standard or separate abbreviations upstream |
| Production loss is zero | Missing/nonpositive production speed | Correct source Tracker study and reload |
| Observed time is unexpected | Manual/segment/event fallback selected | Inspect source basis in Loaded Study Reference |
| Saved analysis reopens with fewer studies | Source records moved, renamed, deleted, or unavailable | Restore source records or rebuild the analysis |
| Data Calculation differs from current Pareto | Different intentional calculation scopes | Compare using equivalent populations and code filters |

---

## 8. Records and Retention

Retain the repository analysis package when the analysis supports an engineering decision, improvement project, or recurring review.

The analysis package should be considered dependent on the underlying source Downtime studies. Do not delete or relocate source records casually if saved analyses must remain reproducible.

---

## 9. Related Documents

- **QS-AE-MSDT-001** — Quick Start
- **REF-AE-MSDT-001** — Downtime Aggregation & Pareto Methodology
- **LIM-AE-MSDT-001** — Limitations
- **DATA-AE-MSDT-001** — Data Handling & Repository Records
