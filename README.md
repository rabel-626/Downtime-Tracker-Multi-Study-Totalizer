# Abel Engineering Multi-Study Downtime Totalizer

Documentation snapshot: September 2, 2026

## Purpose

The Abel Engineering Multi-Study Downtime Totalizer combines multiple saved Downtime Tracker JSON studies into a reason-code Pareto and production-loss analysis. It is intended for cross-study comparison, recurring-loss identification, location or shift filtering, study-level drilldown, and consolidation of similar reason-code tags.

The application is contained in one HTML file. It runs locally in the browser without installation, a web server, a database, or an internet connection.

Current application file:

`ccac1abb-c07a-4608-8837-ac559f38334e.html`

## Recommended browser

- Use current desktop Microsoft Edge or Google Chrome.
- JavaScript must be enabled.
- Edge or Chrome provides the most dependable multi-file selection and folder-selection behavior.
- Folder drag-and-drop support varies by browser. Use **Browse for Study Folder** if a folder drop is not recognized.
- For PDF output, enable **Background graphics** in the print dialog.

## Quick start

1. Open the Totalizer HTML file in Edge or Chrome.
2. Drag saved Tracker JSON files into the file drop zone, select multiple JSON files, or browse for a study folder.
3. Confirm the load-status message reports the expected number of studies.
4. Set location, shift, sort, and Pareto-source filters.
5. Review the reason-code inclusion checkboxes.
6. Create optional grouping rows for codes that represent the same practical loss.
7. Review the Pareto chart and table, production metrics, Data Calculation table, and Loaded Study Reference.
8. Export the grouping CSV if the grouping configuration should be reused.
9. Print the current Pareto view or export the Excel-compatible report.

## Accepted input

The Totalizer accepts local `.json` files containing an `events` array. It is designed for JSON files exported by the Abel Engineering Downtime Tracker, including files that retain the legacy `MPG_Downtime_Tracker_Study` compatibility identifier.

Non-JSON files are rejected. A JSON file that cannot be parsed or does not contain an event array is reported as failed.

The Totalizer reads these source-study fields when available:

- Saved or generated filename
- Location
- Study date
- Shift
- Line or machine
- Product or job
- Observer or operator
- Study objective
- Production speed
- Manual study duration
- Timer study segments
- Downtime events and event timestamps
- Reason-code abbreviations and descriptions

Only events with a numeric duration greater than zero are included.

## Loading studies

Available methods:

- **Select JSON Files** for one or more files.
- Drag individual JSON files into **Drop JSON files here**.
- **Load All JSONs from Folder** or **Browse for Study Folder**.
- Drag a study folder into the folder drop zone when supported by the browser.

The Totalizer does not deduplicate studies. Loading the same JSON file twice counts it twice. Confirm the Loaded Study Reference before using the totals.

## Observed-time precedence

For each study, observed time is determined in this order:

1. A positive manual study duration from the source file.
2. The sum of valid completed timer segments.
3. The elapsed span from the earliest to latest valid event timestamp.
4. Unavailable, resulting in zero observed time.

The Loaded Study Reference identifies the observed-time source used for each study.

## Filters and Pareto source

The Totalizer can filter or slice the analysis by:

- Location
- Shift
- Study ordering
- All visible studies or one individual study
- Included or excluded reason-code tags

The main Pareto metrics, chart, and Pareto table recalculate from the current visible study set and included reason codes.

The study selector can isolate one source JSON to show its individual contribution without unloading the other studies.

## Reason-code inclusion

Every loaded reason-code tag starts included in the Totalizer. Use the reason-code checkboxes to remove planned events, non-loss categories, or other tags that should not contribute to the current Pareto.

Important: the Totalizer does not currently import the Tracker code definition's `includeInSummary` setting. A code excluded in an individual Tracker study is still included by default after loading into the Totalizer. Reapply the intended inclusion selection before relying on combined totals.

## Similar-tag grouping

Grouping rows allow different source codes to be managed as a related analytical group.

### Create a group

1. Enter a group name.
2. Select **Add Grouping Row**.
3. Drag reason-code tags into the row, or check ungrouped tags and use **Add Checked Ungrouped Tags to Last Group**.
4. Select the group's display mode.

### Group display modes

| Mode | Result |
| --- | --- |
| Show this group combined | All member-code events roll into one Pareto contributor named after the group. |
| Show this group as original tags | The row organizes the tags, but the Pareto continues to show each original code separately. |

A reason code can belong to only one group. Moving it to a new group removes it from its prior group.

Group-level controls can show or hide all member tags. Individual grouped tags also retain their own inclusion checkboxes.

### Preserve group mappings

The Totalizer does not automatically persist group definitions between sessions. Use **Export Group CSV** before closing or reloading the page, then use **Import Group CSV** in a later session.

Grouping CSV fields:

- Group Name
- Tag Code
- Group View Mode (`COMBINED` or `SEPARATE`)

Imported tag codes that do not exist in the currently loaded studies are ignored.

## Calculations

### Per-study normalization

| Metric | Current calculation |
| --- | --- |
| Observed time | Manual total, otherwise summed segments, otherwise event timestamp span. |
| Total downtime | Sum of all loaded valid event durations for that study. |
| Projected or optimum units | Production speed × observed hours. |
| Production loss | Production speed × downtime hours. |
| Net available production | Projected units minus production loss, never below zero. |
| Downtime percentage | Downtime divided by observed time. |

### Current filtered Pareto view

| Metric | Current calculation |
| --- | --- |
| Events | Included event count in visible studies. |
| Downtime | Sum of included-event durations in visible studies. |
| Observed time | Sum of full observed time for visible studies. |
| Downtime percentage | Included downtime divided by visible observed time. |
| Optimum production | Production speed × observed hours for visible studies. |
| Production loss | Production speed × included downtime hours for visible studies. |
| Production utilization | `(optimum production - production loss) / optimum production`. |

The Pareto table ranks effective reason codes or combined group names by downtime. It reports description, event count, downtime, downtime minutes, percentage, cumulative percentage, and the number of affected studies.

## Analysis sections

### Combined Downtime Pareto

Shows the current filtered metrics, violet Pareto chart, cumulative-percent curve, expandable reason-code detail, and source-study contribution records.

### Data Calculation Tab

Shows a per-study calculation table and combined total for all loaded studies, including production speed, observed time, downtime, projected production, production loss, net production, and downtime percentage.

Important: this table uses all loaded studies and each study's full normalized downtime. It does not mirror every current Pareto filter, grouping, or tag exclusion. Use it as a raw loaded-study calculation reference.

### Loaded Study Reference

Preserves source context and timing information for each loaded JSON file. Expand a study to review:

- Observed-time source
- Timer segments
- Event numbers
- Original reason codes
- Display times and full timestamps
- Event durations
- Notes

Individual studies can be removed without clearing the entire session.

## Outputs

| Output | Contents |
| --- | --- |
| Print Pareto PDF | Current Pareto source, filters, selected tags, visible-study list, metrics, chart, and Pareto table. |
| Excel-compatible `.xls` | Summary, Pareto breakdown, study reference, data calculation, event timing, segment timing, and tag-group mapping tables. |
| Grouping CSV | Reusable group name, tag code, and view-mode mapping. |

### Excel export scope

The Excel-compatible export intentionally contains several scopes:

- **Summary** reflects the current Pareto view and code inclusion state.
- **Pareto Breakdown** reflects the current calculated Pareto.
- **Study Reference** reflects the visible studies.
- **Data Calculation** includes all loaded studies.
- **Event Timing Records** include visible studies and currently included codes.
- **Study Segment Timing** reflects visible studies.
- **Tag Group Mapping** reflects the current grouping setup.

Review filters, tag inclusion, and loaded-study count before export.

## Session behavior and privacy

All calculations occur locally in the browser. The application does not send study data to an Abel Engineering server or an external data API.

Loaded studies, selected filters, excluded codes, expanded rows, and tag groups are held in page memory. Closing or reloading the page clears the working session. Export the grouping CSV when the mapping should be retained; original Tracker JSON files remain unchanged.

## Current compatibility cautions

### Legacy identifiers

Some generated filenames still use an `MPG_` prefix. These names were retained during the visual-only Abel Engineering rebrand to avoid disrupting established file-handling practices. The interface and visible report title use Abel Engineering branding.

### Source inclusion settings are not imported

The Totalizer starts every observed reason code as included, regardless of the source Tracker's `includeInSummary` value. Reapply exclusions in the Totalizer.

### Categories are not part of the aggregation key

The Totalizer currently aggregates by reason-code abbreviation and does not read downtime categories. Identical abbreviations used under different categories are combined into one reason-code tag. Prefer unique abbreviations across categories when separation is required.

### Two-part manual-duration interpretation

The Tracker accepts two-part manual totals as `[MM]:SS`, but the current Totalizer interprets a two-part source duration as hours and minutes. For cross-tool consistency, use a plain minute value such as `120` or a three-part value such as `2:00:00` when creating manual studies intended for the Totalizer. Avoid relying on `120:00` until the parsing behavior is aligned in a future functional update.

### Repeated files are counted repeatedly

There is no duplicate-file guard. Remove duplicate rows or clear and reload the intended files.

### Missing production speed

A study without a positive production speed can contribute to downtime and Pareto results, but its production projections and loss calculations will be zero or marked missing.

## Troubleshooting

### Files are rejected

- Confirm the files use the `.json` extension.
- Confirm they came from the Downtime Tracker or contain an `events` array.
- Do not select CSV, Excel, or PDF reports in the study loader.

### A folder drop loads nothing

- Use **Browse for Study Folder** instead.
- Confirm the folder contains JSON files.
- Try current desktop Edge or Chrome.

### Downtime totals are higher than expected

- Check for duplicate loaded files.
- Review the reason-code inclusion checkboxes.
- Remember that source Tracker exclusions are not applied automatically.
- Check whether identical abbreviations from different categories were combined.

### Observed time is unexpectedly large

- Expand the study in Loaded Study Reference and inspect the observed-time source.
- For a manual study, inspect the original manual-duration format.
- Replace a two-part duration with plain minutes or `H:MM:SS` in the source Tracker and re-export the JSON.

### Production metrics are zero

- Confirm the source study has a positive production speed.
- Confirm observed time is available.
- Confirm included events have positive durations.

### Group mappings disappear

- Group definitions are session-only unless exported.
- Export the grouping CSV after changes and import it after loading the relevant study files next time.

### PDF styling is incomplete

- Enable **Background graphics** in the print dialog.
- Use landscape orientation.
- Expand the Pareto table before printing if the complete table should be included.

## Branding

The September 2026 visual update applies the Abel Engineering identity throughout the interface and visible report content:

- Embedded Abel Engineering logo
- Carbon-black interface surfaces
- White and silver typography
- Electrical-violet controls, highlights, charts, and gradients

The logo is embedded in the HTML, so no separate image file is required during normal use.

## Change record

### September 2, 2026

- Replaced the previous visual branding with Abel Engineering branding.
- Embedded the supplied logo directly in the standalone HTML file.
- Reworked interface, chart, and visible export colors to the Abel Engineering black, white, silver, and violet palette.
- Preserved study parsing, calculations, controls, legacy filenames, and source compatibility.

## Maintenance notes

- Coordinate schema or duration-parser changes with the Downtime Tracker.
- Preserve compatibility with `MPG_Downtime_Tracker_Study` JSON payloads.
- Add explicit migration handling before changing legacy output names or parsing rules.
- Validate multi-file load, folder load, tag inclusion, combined and separate groups, grouping CSV round-trip, Pareto detail, study detail, Excel output, and PDF output after functional changes.
- Update this README whenever source fields, filter scope, calculation formulas, grouping behavior, exports, or session persistence change.

