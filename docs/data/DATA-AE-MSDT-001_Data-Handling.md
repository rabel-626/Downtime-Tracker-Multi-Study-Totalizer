---
documentId: DATA-AE-MSDT-001
title: Multi-Study Downtime Totalizer Data Handling & Repository Records
tool: Downtime-Tracker-Multi-Study-Totalizer
documentVersion: 1.1
toolVersion: 1.2.0
status: Review Draft
updated: 2026-09-09
---

# Multi-Study Downtime Totalizer — Data Handling & Repository Records

## 1. Purpose

This document describes how the **Abel Engineering Multi-Study Downtime Totalizer** reads source Downtime studies, manages its active browser session, saves reusable Multi-Study analysis records, resolves source references when reopening an analysis, and produces report outputs.

---

## 2. Data Flow Overview

The preferred current workflow is:

```text
Downtime Tracker
    ↓
02_DOWNTIME source study packages
    ↓
Multi-Study Downtime Totalizer
    ↓
Analysis settings + Pareto + production calculations
    ↓
04_MULTI_STUDY_DOWNTIME analysis package
```

The source studies and analysis records are deliberately separate.

The Totalizer does not overwrite the source Downtime Tracker study package when an analysis filter, grouping, or slicer setting changes.

---

## 3. Study Repository Structure

The repository schema uses these standard routes:

```text
01_CYCLE_TIME
02_DOWNTIME
03_TAKT_MATERIAL_FLOW
04_MULTI_STUDY_DOWNTIME
90_TEMPLATES
```

The Totalizer uses:

```text
Source route:   02_DOWNTIME
Analysis route: 04_MULTI_STUDY_DOWNTIME
```

The application recognizes repository roots named:

```text
Engineering Study Hub - Study Repository
```

or:

```text
Study Repository
```

A repository schema file is maintained under:

```text
_SYSTEM/LIBRARY_SCHEMA.json
```

with the repository type:

```text
AbelEngineering.StudyRepository
```

and repository schema version:

```text
1
```

---

## 4. Browser Repository Access

Repository access uses the browser's File System Access capability.

The application can:

- request read/write access to the repository root;
- remember the selected folder handle using IndexedDB;
- re-use the remembered handle when browser permission remains available;
- prompt for permission again when required;
- initialize missing standard repository routes when connecting with write access;
- perform a temporary write probe when a verified write operation is required.

The remembered handle is stored in browser-managed IndexedDB under an Abel Engineering repository-handle database.

> **Important:** Remembering the handle does not guarantee permanent permission. The browser may require the user to authorize the folder again.

---

## 5. Source Study Discovery

### 5.1 Repository source scan

The Totalizer scans recursively under:

```text
02_DOWNTIME
```

for files matching the Downtime study naming pattern:

```text
*__STUDY.json
```

A candidate repository source is accepted by the repository browser when the JSON contains the compatible Downtime Tracker file type:

```text
MPG_Downtime_Tracker_Study
```

The legacy identifier is intentionally retained for cross-version compatibility.

### 5.2 Source metadata used in the browser

The repository source selector uses information such as:

- study location;
- line;
- date;
- shift;
- product;
- source record ID;
- repository relative path.

### 5.3 Legacy local input

Legacy File Load can accept local `.json` files when:

- the extension is `.json`;
- JSON parsing succeeds;
- an `events` array exists.

Legacy local files do not need to exist under the Study Repository.

---

## 6. Source Study Data Read by the Totalizer

Depending on source version, the Totalizer may read:

- `fileType`;
- `savedAt`;
- generated filename;
- repository record information;
- Study Information;
- form-field compatibility values;
- production speed;
- manual study duration;
- study timing segments;
- downtime code definitions;
- code descriptions;
- `includeInSummary` settings;
- event records;
- event categories;
- event durations;
- start/stop timestamps;
- event notes;
- source study-scope summary.

The current Totalizer normalizes this source information into an internal per-study record before aggregation.

---

## 7. Source Scope Preservation

Current source scope is preserved in two ways.

### 7.1 Authoritative source summary

When a compatible source study includes a source-scope summary containing required fields such as included study seconds and included downtime seconds, the Totalizer uses that summary.

The source summary may include:

```text
Total Logged Study Seconds
Excluded Event Seconds
Included Study Seconds
Included Downtime Seconds
Included Uptime Seconds
```

### 7.2 Reconstructed source scope

When the authoritative summary is unavailable, the Totalizer reconstructs scope from:

- observed-time fallback logic;
- downtime-code definitions;
- event categories;
- `includeInSummary` settings;
- event durations.

This allows current source inclusion semantics to survive into multi-study analysis even for compatible studies that do not yet contain the newest source-summary object.

---

## 8. Active Browser Session

The application keeps the current analytical state in browser page memory.

Current state includes:

- loaded normalized studies;
- current Pareto rows;
- expanded Pareto rows;
- expanded study rows;
- Totalizer-excluded reason codes;
- selected study;
- selected location;
- selected shift;
- study sort mode;
- tag groups;
- group view modes;
- analysis repository record ID;
- analysis creation timestamp.

This active state is not automatically a permanent record until **SAVE ANALYSIS** is used.

---

## 9. Analysis Identity

The analysis identity contains:

```text
Analysis Name / Identifier
Organizer
Analysis Date
Description
```

A new repository analysis record is assigned an ID in the form:

```text
MDA-YYYYMMDD-XXXX
```

The record ID is retained for later saves of that active analysis.

---

## 10. Analysis Repository Path

The destination route is constructed from the source-study population and analysis date.

If every loaded source has one common location and line:

```text
04_MULTI_STUDY_DOWNTIME/
└── LOCATION/
    └── LINE/
        └── YYYY/
            └── YYYY-MM/
                └── MDA-RECORD-ID/
```

If multiple locations or lines are represented, the path uses:

```text
MULTI_LOCATION
MULTI_LINE
```

as appropriate.

---

## 11. Saved Analysis Package

**SAVE ANALYSIS** writes three files.

### 11.1 Analysis definition

```text
MDA-...__ANALYSIS.json
```

Current analysis schema:

```text
schema: AbelEngineering.MultiStudyDowntimeAnalysis
schemaVersion: 3
```

The analysis contains:

- application identifier;
- generation timestamp;
- analysis identity;
- creation/update timestamps;
- calculation contract;
- repository identity;
- source filenames;
- structured source-study references;
- current Totalizer settings;
- summary rows.

### 11.2 Excel-compatible report

```text
MDA-...__REPORT.xls
```

This is an HTML-based Excel-compatible report containing the current exported tables.

### 11.3 Package manifest

```text
MDA-...__MANIFEST.json
```

Current manifest contract:

```text
fileType: AbelEngineering.StudyPackageManifest
schemaVersion: 2
recordType: MULTI_STUDY_DOWNTIME
```

The manifest records:

- record ID;
- analysis name;
- organizer;
- analysis date;
- description;
- source files;
- structured source-study references;
- saved timestamp;
- relative path;
- output inventory.

---

## 12. Calculation Contract Stored in the Analysis

Current saved analyses contain:

```text
calculationContract.version = 2
```

with the following declared meanings:

```text
studyTimeBasis:
  source included study seconds

potentialProduction:
  production speed × included study hours

estimatedLostProduction:
  production speed × included downtime hours

estimatedAvailableProduction:
  production speed × included uptime hours
```

This metadata is important because it records how the analysis interprets the source Downtime studies.

---

## 13. Source Study References in a Saved Analysis

Each source reference can contain:

```text
fileName
repositoryRecordId
repositoryRelativePath
repositoryFilePath
location
date
shift
line
product
observer
includedStudySeconds
totalDowntimeSeconds
productionSpeed
```

The structured repository references are more reliable than filenames alone when reopening an analysis.

---

## 14. Reopening a Saved Analysis

**OPEN SAVED ANALYSIS** scans:

```text
04_MULTI_STUDY_DOWNTIME
```

for:

```text
*__ANALYSIS.json
```

with schema:

```text
AbelEngineering.MultiStudyDowntimeAnalysis
```

After the user selects an analysis, the Totalizer scans `02_DOWNTIME` and attempts to resolve each saved source reference.

Current source-resolution precedence is:

```text
1. repositoryRecordId
2. repositoryRelativePath
3. repositoryFilePath
4. fileName
```

The first unused matching repository record is selected for each saved source reference.

---

## 15. Missing Sources During Reopen

If some sources resolve and others do not:

- the analysis opens with the available source studies;
- missing source studies are reported in a warning;
- saved filters/groups are reapplied to the available population.

If **no** required source study can be resolved:

- the analysis cannot be reconstructed from the repository;
- the application raises an error.

> **Records-management implication:** A saved analysis is dependent on the continued availability and identity of its source studies.

---

## 16. Restored Analysis Settings

When a saved analysis opens, the application restores:

- Totalizer-excluded reason codes;
- tag groups;
- tag group view modes;
- selected study ID when still valid;
- selected location;
- selected shift;
- sort mode;
- analysis repository record ID;
- analysis identity;
- original analysis creation timestamp.

The application then recalculates the analysis from the currently resolved source-study records.

This is preferable to storing only frozen summary numbers because source-event drilldown remains available.

---

## 17. Save Verification Behavior

The repository writer performs several local checks during save.

For general outputs it verifies that the saved file size matches the generated blob size.

For JSON outputs it additionally:

- reads the file back;
- confirms valid JSON parsing;
- confirms the stored JSON text matches the generated JSON;
- calculates a SHA-256 hash when browser cryptographic support is available.

The Analysis JSON output is recorded in the manifest output inventory with its hash when available.

> **Scope:** This verifies the local write operation performed by the Totalizer. Synchronization to OneDrive/SharePoint is performed by the external synchronization client and is not the same operation.

---

## 18. Excel-Compatible Report Contents

The current report includes:

### Summary

Current analysis identity and active Pareto headline metrics.

### Pareto Breakdown

The current calculated effective Pareto contributors.

### Study Reference

Visible source-study context and source normalized calculations.

### Data Calculation

All loaded source-study normalized calculations.

### Event Timing Records

Event-level records for visible studies and currently Totalizer-included codes.

### Source-Excluded Event Audit

Events preserved from source studies but omitted by source `Include in Summary` logic.

### Study Segment Timing

Source timer-segment records for visible studies.

### Tag Group Mapping

Current group names, source codes, and group display mode.

Because these sheets intentionally use different analytical scopes, the user should not assume every table total must be identical.

---

## 19. Grouping CSV

The standalone grouping export/import workflow remains available for reusable mapping portability.

Current imported format can recognize:

```text
Group Name
Tag Code
Group View Mode
```

The current direct grouping CSV export may contain the core group/tag mapping, while saved repository analyses preserve the full tag-group configuration including group mode directly inside the Analysis JSON.

For repeatable repository-based work, **SAVE ANALYSIS** is the preferred method because it preserves source references and the complete active analysis state together.

---

## 20. Legacy Files

Some compatibility identifiers retain the historic `MPG_` prefix.

This includes source Downtime Tracker file type:

```text
MPG_Downtime_Tracker_Study
```

The identifier is a compatibility contract, not a statement of current branding.

Do not manually rename schema/file-type values inside saved JSON files unless a coordinated migration is implemented across all dependent tools.

---

## 21. Privacy and Network Behavior

The analytical calculations execute in the browser.

The repository workflow writes directly to the user-selected local/synchronized Study Repository through browser file access.

The application does not require an Abel Engineering backend API to perform the analysis.

OneDrive/SharePoint or another synchronization client may subsequently synchronize the files because the selected repository folder itself is synchronized.

### Data leaves the active browser session when the user intentionally:

- saves an analysis to the Study Repository;
- exports the Excel-compatible report;
- prints/saves a PDF;
- exports grouping CSV;
- loads or saves files through user-selected storage.

---

## 22. Data Protection Guidance

Recommended practices:

- preserve original source Downtime study packages;
- avoid editing source JSON manually;
- use analysis names that identify scope without exposing unnecessary sensitive details;
- avoid entering personal employee information in notes where aliases or roles are sufficient;
- retain repository folder permissions appropriate to the operational data;
- verify OneDrive/SharePoint synchronization before assuming another user has the latest analysis;
- do not rely on a report `.xls` as a substitute for the saved Analysis JSON and source studies.

---

## 23. Backup and Recovery

For a repository-based analysis, the critical recovery set is:

```text
1. Source Downtime study records under 02_DOWNTIME
2. Multi-Study Analysis JSON under 04_MULTI_STUDY_DOWNTIME
3. Associated package manifest
```

The generated report can be recreated from the analysis if the application and all source records remain available.

If source studies are lost, the saved Analysis JSON alone does not embed their full event history and cannot fully reconstruct the original analysis drilldown.

---

## 24. Compatibility and Change Control

Future changes to any of the following require coordinated validation:

- Downtime Tracker source schema;
- source inclusion logic;
- duration parsing;
- source repository routes;
- Multi-Study analysis schema;
- source-resolution identity fields;
- reason-code aggregation key;
- calculation contract;
- grouping behavior;
- export scopes.

At minimum, validation should include:

- repository source load;
- legacy source load;
- source exclusion preservation;
- observed-time derivation;
- reason-code filter behavior;
- combined/separate group behavior;
- Pareto totals;
- production-impact totals;
- save/open round trip;
- missing-source handling;
- Excel export;
- Pareto print output.

---

## 25. Related Documents

- **QS-AE-MSDT-001** — Quick Start
- **WI-AE-MSDT-001** — Analysis Procedure
- **REF-AE-MSDT-001** — Downtime Aggregation & Pareto Methodology
- **LIM-AE-MSDT-001** — Limitations
