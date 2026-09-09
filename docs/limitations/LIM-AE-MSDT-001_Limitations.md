---
documentId: LIM-AE-MSDT-001
title: Multi-Study Downtime Totalizer Limitations
tool: Downtime-Tracker-Multi-Study-Totalizer
documentVersion: 1.1
toolVersion: 1.2.0
status: Review Draft
updated: 2026-09-09
---

# Multi-Study Downtime Totalizer — Limitations

## Intended Use

The **Abel Engineering Multi-Study Downtime Totalizer** is an engineering analysis aid for combining Downtime Tracker studies, constructing downtime Pareto views, comparing recurring losses, estimating production impact, and preserving repeatable multi-study analysis configurations.

It should be treated as a structured analytical layer over the source Downtime Tracker records—not as an independent source of observed production facts.

---

## 1. Source Data Quality Controls the Result

The Totalizer cannot correct inaccurate source observations automatically.

Errors in any source study can propagate into the combined analysis, including:

- missed downtime starts or stops;
- incorrect event duration;
- incorrect reason code;
- vague or inconsistent notes;
- wrong production speed;
- incorrect study duration;
- incorrect `Include in Summary` configuration;
- duplicate source records;
- inconsistent coding standards between lines or periods.

A mathematically correct Pareto can still be operationally misleading when the source data is inconsistent.

---

## 2. Repository Loading Does Not Independently Revalidate the Entire Source Package

The repository loader scans `02_DOWNTIME` for compatible Downtime Tracker study JSON files and validates the source study type expected by the Totalizer.

It does **not** independently re-run every source Tracker data-quality check, nor does the current loading path reconstruct a complete end-to-end package-validation audit before analysis.

Therefore:

- a source study being visible in the repository does not prove that the study itself was fully reviewed;
- the analyst remains responsible for source-study quality;
- source package status should be reviewed separately when formal verification is required.

---

## 3. Duplicate Studies Are Not Automatically Prevented

The Totalizer does not maintain a hard duplicate-study guard across the active session.

If the same study is loaded twice:

- event count doubles;
- downtime doubles;
- included study time doubles;
- production opportunity doubles;
- estimated production loss doubles;
- Pareto contribution doubles.

Always review the Loaded Study Reference before finalizing a result.

---

## 4. Reason-Code Semantics Must Be Consistent

The Pareto aggregation key is primarily the normalized reason-code abbreviation.

Category is not part of the active aggregation key.

Therefore:

```text
Robot / JAM
Feeder / JAM
```

are both aggregated as:

```text
JAM
```

if both source events use the abbreviation `JAM`.

This can incorrectly combine unrelated losses when local coding standards reuse abbreviations.

### Control

Use unique reason-code abbreviations across semantically different losses, or reconcile the source coding standard before performing cross-study analysis.

---

## 5. Grouping Is an Analyst-Defined Interpretation

Tag grouping does not discover semantic equivalence automatically.

When an analyst combines codes into a group, the application assumes the grouping decision is valid.

A group can become misleading when:

- superficially similar codes have different root causes;
- one code is planned and another is unplanned;
- one code represents upstream starvation while another represents equipment failure;
- the codes require different owners or corrective actions.

Grouping should be documented and reviewed.

---

## 6. Current Source Inclusion and Analysis Inclusion Are Different Layers

The current Totalizer preserves source Downtime Tracker inclusion/exclusion logic and then applies a second Totalizer analysis filter.

This produces two legitimate but different questions:

### Source scope

> What downtime did the originating Tracker study define as included?

### Active Totalizer scope

> Which of those source-included codes do I want in this particular analysis?

Filtering out a code in the Totalizer does not restore that duration as uptime and does not alter the source study.

Users must not confuse an active filtered view with the complete source-study downtime model.

---

## 7. Data Calculation and Pareto Use Different Scopes

The **Data Calculation** table uses all loaded studies and each source study's normalized included downtime.

The active **Pareto** responds to:

- visible-study filters;
- individual-study selection;
- Totalizer reason-code inclusion;
- grouping.

As a result, the two sections may legitimately show different downtime totals and percentages.

This is not necessarily a defect.

> **Control:** Compare scopes before comparing numbers.

---

## 8. Observed-Time Fallbacks Can Be Approximate

When an authoritative source scope summary is unavailable, the Totalizer may derive logged study time from:

1. manual duration;
2. timer segments;
3. event timestamp span.

The event timestamp span is particularly limited because:

```text
latest event timestamp − earliest event timestamp
```

does not prove continuous active observation over the entire interval.

Use fallback-derived studies cautiously in cross-study percentage comparisons.

---

## 9. Two-Part Manual Duration Has a Cross-Tool Compatibility Risk

In the current Totalizer fallback parser, a two-part duration is interpreted as:

```text
H:MM
```

The Downtime Tracker has historically accepted two-part manual totals as:

```text
[MM]:SS
```

Therefore a source value such as:

```text
120:00
```

can be interpreted differently if the Totalizer must fall back to the raw manual-duration field.

### Control

Prefer:

- current source files containing authoritative scope summaries;
- plain numeric minutes for legacy files; or
- unambiguous `H:MM:SS` formatting.

---

## 10. Production Loss Is a Linear Estimate

The production model assumes:

```text
Estimated Lost Units
= Production Speed × Downtime Hours
```

This assumes a stable, representative production speed.

It does not automatically model:

- variable machine speed;
- ramp-up or restart losses beyond the captured downtime;
- catch-up production;
- blocked/starved interactions between processes;
- production buffers;
- product mix changes;
- scrap or rework interaction;
- labor constraints;
- non-linear process recovery;
- scheduled breaks unless explicitly represented in the source scope.

The result is an engineering estimate of production opportunity, not a guarantee of recoverable output.

---

## 11. Missing Production Speed Does Not Remove Downtime

A source study without a positive production speed can still contribute:

- events;
- downtime;
- Pareto share;
- study occurrence count.

However, its production-opportunity and production-loss metrics are zero or unavailable.

Therefore, production-impact totals can understate the impact of the loaded study population when some source studies lack a valid speed reference.

---

## 12. Filtered Availability Is Not Full Availability

In the active filtered Pareto view:

- potential production is based on the full included study time of the visible source studies;
- lost production is recalculated only from the currently included Totalizer reason codes.

Thus:

```text
Filtered Availability
= (Potential Production − Filtered Loss)
  / Potential Production
```

If only one loss family is selected, this value answers:

> “What would availability look like if only this selected loss family were counted?”

It does not represent the source study's complete downtime availability.

---

## 13. The Tool Does Not Calculate OEE

The Totalizer's availability-style metrics should not be called OEE unless the required OEE model is established separately.

OEE requires deliberate treatment of:

- Availability;
- Performance;
- Quality;
- planned production time and exclusions;
- ideal-cycle basis.

The Totalizer primarily analyzes captured downtime and production opportunity from the configured source scope.

---

## 14. Pareto Dominance Does Not Prove Root Cause

A Pareto identifies where the largest measured losses accumulate.

It does not prove:

- causal mechanism;
- failure mode;
- ownership;
- corrective action;
- economic return;
- statistical significance.

Use Pareto ranking to prioritize investigation, then verify with event notes, process observation, maintenance records, controls data, quality data, and root-cause analysis as appropriate.

---

## 15. Study Frequency Can Bias Cross-Study Totals

A line, shift, or condition observed more often will naturally contribute more total event time to a simple combined Pareto.

For example:

```text
Line A = 20 hours observed
Line B = 2 hours observed
```

A raw downtime total will favor Line A simply because it has ten times the observation exposure.

The Totalizer provides observed-time denominators and study-level calculations, but it does not automatically transform every Pareto into an exposure-normalized rate comparison.

When comparing unlike exposure periods, consider:

- downtime percentage;
- events per hour calculated separately if needed;
- normalized study rates;
- equal-duration sampling designs;
- statistical modeling outside the Totalizer.

---

## 16. Distinct Studies Affected Is Not a Frequency Rate

`Studies Affected` counts distinct source studies containing at least one active event for the Pareto contributor.

It does not account for:

- study duration;
- study count differences between locations;
- production volume;
- exposure opportunity;
- number of shifts represented.

Use it as a recurrence indicator, not a formal incidence rate.

---

## 17. Saved Analyses Depend on Source Records

A saved repository analysis stores references to its source Downtime studies.

When reopening, the application attempts to resolve those sources from `02_DOWNTIME`.

If source records are:

- deleted;
- moved;
- renamed;
- unavailable due to synchronization;
- otherwise no longer resolvable;

the analysis may reopen with missing studies or fail if none can be resolved.

The saved Analysis JSON is therefore not a complete embedded copy of every source Downtime study.

---

## 18. Analysis IDs Persist Across Re-Saves

Once a repository analysis record ID is assigned, subsequent saves of the same active analysis reuse that record path.

This is useful for revision-in-place behavior, but users should understand that **SAVE ANALYSIS** is not automatically a new immutable analysis record every time.

Create a logically new analysis when a separate historical record is required.

---

## 19. Browser and File-System Limitations

Repository access depends on browser support for the File System Access API.

The full repository workflow is intended for desktop Edge/Chrome.

Potential limitations include:

- browser permission expiration;
- remembered folder handle requiring reauthorization;
- OneDrive/SharePoint synchronization delay;
- offline sync conflicts;
- unsupported mobile directory access;
- local browser policy restrictions.

The application writes locally to the synchronized Study Repository; OneDrive or other sync software performs the cloud synchronization separately.

---

## 20. Excel Export Is HTML-Based `.xls`

The Excel-compatible report is generated as HTML content with an `.xls` filename and Excel-compatible MIME type.

It is intended for convenient spreadsheet review, but it is not a native `.xlsx` workbook.

Possible consequences include:

- Excel compatibility warnings;
- formatting variation by spreadsheet application;
- different import behavior in non-Microsoft software.

The saved Analysis JSON is the reusable analytical configuration; the `.xls` file is a report output.

---

## 21. PDF Output Depends on the Browser Print Engine

Pareto PDF output uses browser printing.

Results can vary with:

- browser;
- paper size;
- scale;
- margins;
- background-graphics setting;
- OS print driver.

Use landscape orientation and enable **Background graphics** for the intended visual output.

---

## 22. Privacy and Confidentiality

The application processes data locally in the browser and writes user-directed files to the selected repository or export destination.

However, local processing does not remove organizational responsibility for:

- sensitive production information;
- employee-identifying notes;
- proprietary product information;
- controlled operational records;
- access permissions to synchronized repositories.

Use non-sensitive identifiers where practical and follow site/company data-handling requirements.

---

## 23. Engineering Use Disclaimer

Abel Engineering tools are provided as engineering analysis and planning aids. Results depend on source data, observations, assumptions, configuration, and analyst-selected scope and should be independently reviewed before being used for production, staffing, financial, regulatory, safety, or equipment-design decisions.

Users are responsible for verifying source studies, calculations, filter scope, group mappings, outputs, and suitability for their intended application.

The Totalizer does not replace professional engineering judgment, applicable standards, manufacturer requirements, site procedures, safety reviews, statistical validation, or formal root-cause analysis.

---

## Related Documents

- **QS-AE-MSDT-001** — Quick Start
- **WI-AE-MSDT-001** — Analysis Procedure
- **REF-AE-MSDT-001** — Downtime Aggregation & Pareto Methodology
- **DATA-AE-MSDT-001** — Data Handling & Repository Records
