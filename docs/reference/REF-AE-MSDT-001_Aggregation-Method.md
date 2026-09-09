---
documentId: REF-AE-MSDT-001
title: Multi-Study Downtime Aggregation & Pareto Methodology
tool: Downtime-Tracker-Multi-Study-Totalizer
documentVersion: 1.1
toolVersion: 1.2.0
status: Review Draft
updated: 2026-09-09
---

# Multi-Study Downtime Totalizer — Aggregation & Pareto Methodology

## 1. Purpose

This reference documents the current analytical model used by the **Abel Engineering Multi-Study Downtime Totalizer**.

It explains how the application:

- interprets source Downtime Tracker scope;
- preserves source inclusion/exclusion decisions;
- determines included study time;
- normalizes production impact;
- applies study and code filters;
- groups related tags;
- constructs the Pareto;
- calculates current-view production metrics;
- distinguishes the filtered Pareto from the raw Data Calculation table.

The equations below describe the application's present calculation contract and are intended for engineering review and validation.

---

## 2. Source Study Model

Each loaded source study is normalized into a common internal structure containing, when available:

- filename and generated filename;
- location;
- date;
- shift;
- line / machine;
- product / job;
- observer;
- study objective;
- production speed;
- source scope summary;
- study timing segments;
- source-included events;
- source-excluded events;
- reason-code descriptions;
- source repository identity.

Only event records with a positive duration ultimately contribute to the active Pareto.

---

## 3. Source Inclusion Contract

### 3.1 Current behavior

The current application preserves the Downtime Tracker's source inclusion state.

If a source study supplies an authoritative study-scope summary, the Totalizer uses that summary as the primary source-scope contract.

If it does not, the Totalizer reconstructs source inclusion from the source study's downtime-code definitions.

A code definition may contain:

```text
Reason Code
Category
Include in Summary
Description
```

When matching a source event to a code definition, the Totalizer:

1. normalizes the reason-code abbreviation;
2. uses the event category to locate the exact definition when possible;
3. otherwise falls back to the first definition for that abbreviation;
4. treats the event as source-included unless the matching definition explicitly has `includeInSummary = false`.

Unknown or unassigned codes are treated as source-included by default.

### 3.2 Source event partitions

Let:

```text
E = all completed source events with numeric duration
I = source-included events
X = source-excluded events
```

Then:

```text
E = I ∪ X
```

and:

```text
I ∩ X = ∅
```

Source-excluded events are retained for audit but do not enter the source-included downtime model.

---

## 4. Total Logged Study Time

When the authoritative source summary is unavailable, the application determines total logged study time using the following precedence.

### 4.1 Manual total

If a positive manual study duration is present, it is used first.

Accepted current parsing behavior includes:

```text
120        → 120 minutes
2:00:00    → 2 hours
```

Current two-part parsing in the Totalizer is:

```text
H:MM
```

rather than the Downtime Tracker's `[MM]:SS` manual-entry convention.

This is a compatibility limitation for legacy/fallback reconstruction and should be avoided by using current source scope summaries or unambiguous three-part durations.

### 4.2 Timer segments

If no positive manual total is available and completed segments exist:

```text
T_logged
= Σ MAX(0, Segment Stop Timestamp − Segment Start Timestamp)
```

### 4.3 Event timestamp span

If neither manual total nor usable segments are available:

```text
T_logged
= Latest Valid Event Timestamp − Earliest Valid Event Timestamp
```

This is a fallback estimate of the observation span.

### 4.4 Unavailable

If none of these sources are usable:

```text
T_logged = 0
```

---

## 5. Source Study Scope

When reconstructed from source records:

### 5.1 Excluded event time

```text
T_excluded
= Σ duration(x),  x ∈ X
```

### 5.2 Included study time

```text
T_included
= MAX(0, T_logged − T_excluded)
```

This follows the Downtime Tracker's scope model: source-excluded downtime is removed from the analytical denominator rather than reclassified as uptime.

### 5.3 Included downtime

```text
D_source
= Σ duration(i),  i ∈ I
```

### 5.4 Included uptime

```text
U_source
= MAX(0, T_included − D_source)
```

### 5.5 Source availability

```text
A_source
= U_source / T_included
```

for:

```text
T_included > 0
```

Otherwise availability is reported as zero.

---

## 6. Source Production Model

Let:

```text
R = production speed [units/hour]
```

### 6.1 Potential production

```text
P_potential
= R × T_included / 3600
```

### 6.2 Estimated lost production

```text
P_lost_source
= R × D_source / 3600
```

### 6.3 Estimated available production

```text
P_available_source
= R × U_source / 3600
```

Because:

```text
U_source = T_included − D_source
```

then ideally:

```text
P_available_source
= P_potential − P_lost_source
```

subject to nonnegative clamping and numeric rounding in presentation.

> **Engineering assumption:** These equations model production loss linearly from a single units/hour reference. They do not model ramp-up, backlog recovery, starvation/blocking propagation, speed changes within the study, or nonlinear equipment behavior.

---

## 7. Study Population Filters

The active Pareto is calculated from the **visible study set**.

A source study is visible when it satisfies:

```text
Selected Location
AND
Selected Shift
AND
Selected Individual Study (if any)
```

The sort mode changes display order only; it does not change the included population.

Let:

```text
S_visible = current visible source-study set
```

---

## 8. Analysis-Layer Reason-Code Inclusion

After source inclusion is applied, the Totalizer applies its own current analysis filter.

Let:

```text
C_included = reason codes currently selected in the Totalizer
```

An event contributes to the active Pareto when:

```text
event ∈ source-included events
AND event.study ∈ S_visible
AND event.code ∈ C_included
```

This is important because the active Pareto can intentionally represent only a subset of the source-included downtime.

---

## 9. Reason-Code Identity

The active aggregation key is the normalized reason-code abbreviation.

```text
Code Key = trimmed source reason-code abbreviation
```

Blank codes are normalized to:

```text
UNASSIGNED
```

### Category behavior

Category is preserved on source events and may be used to resolve the source code definition for source inclusion. However, category is **not part of the Pareto aggregation key**.

Therefore:

```text
Category A / JAM
Category B / JAM
```

both aggregate under:

```text
JAM
```

This is why unique abbreviations across semantically different category/code definitions are strongly preferred.

---

## 10. Tag Grouping

### 10.1 One-group rule

A source reason code can belong to only one analytical group.

When a code is moved to another group, it is removed from its previous group.

### 10.2 Combined group

For a combined group `G` containing source codes:

```text
G = {c1, c2, ..., cn}
```

the effective Pareto code is:

```text
EffectiveCode(c) = GroupName(G)
```

for every:

```text
c ∈ G
```

Group downtime becomes:

```text
D_G
= Σ duration(event)
  for all active events whose source code ∈ G
```

Event count becomes:

```text
N_G
= count(active events whose source code ∈ G)
```

Studies affected becomes the number of distinct source-study IDs contributing at least one active event to the group.

### 10.3 Separate group

When group mode is `SEPARATE`, membership is organizational only:

```text
EffectiveCode(c) = c
```

The original reason codes remain independent Pareto contributors.

---

## 11. Pareto Construction

For every active event:

1. identify its effective code;
2. add duration to that effective code;
3. increment event count;
4. add source-study ID to the contributor's distinct-study set;
5. preserve event-level detail for drilldown.

Let the resulting contributors be sorted by downtime:

```text
D1 ≥ D2 ≥ ... ≥ Dn
```

Total active Pareto downtime is:

```text
D_total
= Σ(k=1..n) Dk
```

### 11.1 Contributor percentage

```text
Percent_i
= Di / D_total
```

### 11.2 Cumulative percentage

```text
Cumulative_i
= Σ(k=1..i) Dk / D_total
```

### 11.3 Studies affected

```text
StudiesAffected_i
= COUNT(DISTINCT source study ID contributing to i)
```

This metric is useful for separating a chronic cross-study loss from one isolated long-duration event.

---

## 12. Active Pareto Headline Metrics

For the visible source-study set and current reason-code inclusion:

### 12.1 Events

```text
N_active
= count of active included events
```

### 12.2 Active downtime

```text
D_active
= Σ duration(active included events)
```

### 12.3 Visible included study time

```text
T_visible
= Σ T_included,s
  for s ∈ S_visible
```

Note that reason-code filtering does **not** reduce `T_visible`.

### 12.4 Active downtime percentage

```text
Downtime%_active
= D_active / T_visible
```

for positive `T_visible`.

---

## 13. Current-View Production Metrics

The production view intentionally uses a fixed opportunity denominator for the visible studies while recalculating lost production from the active reason-code selection.

### 13.1 Visible potential production

```text
P_potential,visible
= Σ P_potential,s
  for s ∈ S_visible
```

### 13.2 Filtered lost production

For each visible study:

```text
D_active,s
= Σ duration(active included events in study s)
```

```text
P_lost,active,s
= R_s × D_active,s / 3600
```

Then:

```text
P_lost,active
= Σ P_lost,active,s
```

### 13.3 Filtered available production

```text
P_available,active
= MAX(0, P_potential,visible − P_lost,active)
```

### 13.4 Filtered availability

```text
A_active
= P_available,active / P_potential,visible
```

This means an analyst can answer questions such as:

> “What fraction of the available production opportunity is being lost specifically to the currently selected loss family?”

It should not be interpreted as a complete production-efficiency metric when reason codes have been intentionally filtered out.

---

## 14. View Share

The study slicer also calculates a view-share metric.

Let:

```text
D_visible = active included downtime in visible studies
D_loaded  = active included downtime across all loaded studies
            using the same current code-inclusion state
```

Then:

```text
View Share
= D_visible / D_loaded
```

This indicates how much of the currently selected downtime population is represented by the visible study slice.

---

## 15. Data Calculation Table Contract

The **Data Calculation** table is not the same scope as the active Pareto.

For each loaded source study, it uses the source normalized values:

```text
Included Study Time
Source Included Downtime
Potential Production
Source Estimated Lost Production
Source Estimated Available Production
Source Downtime %
```

Source downtime percentage is:

```text
Downtime%_source
= D_source / T_included
```

The combined total row is:

```text
T_combined = Σ T_included,s
```

```text
D_combined = Σ D_source,s
```

```text
P_potential,combined = Σ P_potential,s
```

```text
P_lost,combined = Σ P_lost_source,s
```

```text
P_available,combined = Σ P_available_source,s
```

```text
Downtime%_combined
= D_combined / T_combined
```

It uses **all loaded studies** rather than the current visible-study slice and does not recalculate from Totalizer reason-code filters or grouping.

This distinction is deliberate.

---

## 16. Saved Analysis Calculation Contract

Current repository analysis records use:

```text
Schema: AbelEngineering.MultiStudyDowntimeAnalysis
Schema Version: 3
Calculation Contract Version: 2
```

The calculation contract explicitly records:

```text
Study time basis:
  source included study seconds

Potential production:
  production speed × included study hours

Estimated lost production:
  production speed × included downtime hours

Estimated available production:
  production speed × included uptime hours
```

This makes the saved analysis self-describing and helps protect against silent interpretation changes in future versions.

---

## 17. Worked Example

Assume two source studies after source exclusions are applied.

### Study A

```text
Included Study Time = 2.00 hr
Production Speed    = 1,200 units/hr
Included Downtime   = 0.20 hr
```

Then:

```text
Potential Production
= 1,200 × 2.00
= 2,400 units
```

```text
Estimated Lost Production
= 1,200 × 0.20
= 240 units
```

```text
Included Uptime
= 2.00 − 0.20
= 1.80 hr
```

```text
Estimated Available Production
= 1,200 × 1.80
= 2,160 units
```

### Study B

```text
Included Study Time = 1.50 hr
Production Speed    = 1,000 units/hr
Included Downtime   = 0.15 hr
```

```text
Potential Production = 1,500 units
Estimated Lost Production = 150 units
Estimated Available Production = 1,350 units
```

### Combined source calculation

```text
Combined Included Study Time
= 2.00 + 1.50
= 3.50 hr
```

```text
Combined Source Downtime
= 0.20 + 0.15
= 0.35 hr
```

```text
Combined Downtime %
= 0.35 / 3.50
= 10.0%
```

```text
Combined Potential Production
= 2,400 + 1,500
= 3,900 units
```

```text
Combined Estimated Lost Production
= 240 + 150
= 390 units
```

If the active Pareto filters to a subset of codes representing only 0.12 hr in Study A and 0.05 hr in Study B, the **filtered loss** becomes:

```text
Study A filtered loss
= 1,200 × 0.12
= 144 units
```

```text
Study B filtered loss
= 1,000 × 0.05
= 50 units
```

```text
Total filtered loss
= 194 units
```

Potential production remains:

```text
3,900 units
```

so filtered availability is:

```text
(3,900 − 194) / 3,900
≈ 95.0%
```

That **95.0%** is the availability after considering only the currently selected loss family. It is not the full source-study availability.

---

## 18. Interpretation Guidance

### Appropriate conclusions

The Totalizer is well suited to statements such as:

- “These three codes account for 67% of the active downtime in the selected study population.”
- “This loss family appears in 8 of 10 source studies.”
- “The selected source population contains 12.4 hours of included study time.”
- “At the source production rates, the selected loss family corresponds to an estimated 4,200 units of production opportunity.”

### Conclusions requiring additional evidence

The Totalizer alone does not prove:

- causal mechanism;
- statistical significance between shifts;
- future production improvement after eliminating a code;
- true OEE;
- equipment reliability distributions;
- labor causality;
- economic savings;
- recoverability of all estimated lost units.

Use the event drilldown, source studies, process knowledge, and appropriate statistical/engineering analysis before making those claims.

---

## 19. Related Documents

- **QS-AE-MSDT-001** — Quick Start
- **WI-AE-MSDT-001** — Analysis Procedure
- **LIM-AE-MSDT-001** — Limitations
- **DATA-AE-MSDT-001** — Data Handling & Repository Records
