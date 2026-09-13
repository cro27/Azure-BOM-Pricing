# Excel Output and Validation Specification

## File

Use:

```text
Azure_Cost_Estimate_<workload>_YYYYMMDD_HHMM.xlsx
```

Use the user's local timezone. Exclude taxes.

Before creating any durable `.xlsx`, require the user to explicitly select the output folder.
Do not default to the current directory, source directory, workspace, OneDrive, a prior
destination, or any other inferred path. A temporary build file is transient staging only and
may be created after the destination gate has been satisfied.

For OneDrive or SharePoint-synchronised folders, build and validate in a non-synchronised
temporary directory first, then copy the validated bytes into the destination. If the
environment applies MIP encryption after the copy, validate opening with the native
Office application where available.

## Required worksheets

### Summary

Include:

- workload and source;
- intended audience;
- estimate maturity and completeness score;
- generated timestamp;
- PAYG retail pricing basis;
- currency and regions;
- price source/retrieval date;
- tax exclusion;
- monthly and annual totals;
- separate Azure and non-Azure Microsoft licence subtotals, plus an optional combined
  estimate;
- totals by environment and service;
- priced, assumed and unpriced item counts;
- high/medium/low confidence totals.

### Estimate Readiness

Include:

- Architecture Extraction Preview;
- completeness score by section and total;
- maturity: Indicative, Qualified or Customer-ready;
- blocker status;
- unresolved inputs ordered by materiality;
- source-extraction confidence;
- age of the price snapshot.

### Cost Detail

One row per billable component:

- environment;
- workload;
- source BOM row/resource identifier;
- scenario (`Baseline`, `Optimised`, `Small`, `Medium` or `Large`) when applicable;
- service;
- region/traffic zone;
- SKU/tier;
- component/meter;
- quantity;
- unit price;
- unit;
- monthly usage;
- monthly cost;
- annual cost;
- pricing basis;
- confidence;
- AHUB treatment;
- assumption;
- selected meter ID.

### Assumptions & Gaps

For every assumption:

- area;
- original missing or conflicting input;
- value/design selected;
- reason;
- materiality;
- user-confirmed or assistant-assumed;
- confidence impact.

If Fabric F2-F32 capacity and Power BI usage are present, include a prominent licensing
gap showing:

- distinct creators and viewers;
- Pro versus PPU decision;
- whether licences are already owned;
- the current official country-localised per-user list price, source URL and retrieval
  date;
- that the licence cost is outside the Azure Retail Prices API and excluded from the
  Azure subtotal.

### Recommendations

Keep recommendations outside the estimate total unless the user approved them.
Classify each by priority and WAF pillar. Include indicative cost impact only when based
on a live or clearly cited meter.

### Scenario Comparison

Include when optimisation was requested. Show Baseline PAYG and each approved Optimised
PAYG scenario with separate monthly and annual totals, delta, assumptions and trade-offs.
Never combine scenarios into the Summary total.

### T-Shirt Sizing

Include when the user explicitly approved T-shirt pricing:

- Small, Medium and Large profile assumptions;
- selected SKU, quantity and usage for each service and size;
- separate monthly and annual totals for each size;
- service-level cost breakdown for each size;
- security, resilience and backup posture applied consistently;
- confidence and the inputs needed to replace each assumption.

T-shirt mode must be labelled Indicative. Do not show one combined total.

### Price Lineage

For every selected price include the API/source fields from API-GUIDE.md, the query URL
or filter, and retrieval timestamp. This sheet is the audit trail.

For non-Azure Microsoft licensing, include the official page URL, country/locale,
displayed currency, billing cadence, tax statement, displayed unit price and retrieval
timestamp.

## Workbook quality

- Freeze header rows and apply filters to detail tables.
- Use AUD/USD/etc. number formats without adding tax.
- Wrap notes and size columns for readability.
- Highlight assumptions, unpriced items and high-priority recommendations.
- Highlight readiness blockers and state when an estimate is not customer-ready.
- Avoid merged cells in filterable data tables.
- Use formulas or a programmatic cross-foot so the summary equals detail.
- Write all source-derived text as literal cells. For text beginning with `=`, `+`, `-`, `@`,
  TAB or CR, prefix a single quote before writing so spreadsheet software cannot interpret it
  as a formula. This applies to identifiers, labels, assumptions, notes, URLs and any other
  text extracted from user-supplied content. Validated numeric fields remain numeric values.
- Allow formulas only when they come from trusted workbook-generation logic with fixed,
  reviewed formula templates. Never copy a source-derived string into a formula cell.

## Validation

Validation is mandatory:

1. save the workbook;
2. verify the file exists and is non-empty;
3. verify the OOXML ZIP structure and CRC;
4. reload with the workbook library used to create it;
5. verify all expected worksheets exist;
6. verify detail totals equal the summary total;
7. scan formulas for errors or broken references;
8. scan every source-derived cell and confirm no untrusted formula cell remains; values that
   originally began with `=`, `+`, `-`, `@`, TAB or CR must be stored as neutralised literal
   text, and every remaining formula must match trusted generation logic;
9. open read-only with Excel COM on Windows when available;
10. after copying to the requested folder, confirm it still opens.

If Excel COM is unavailable, use ZIP/XML validation plus a reload with a second compatible
library where practical. State the actual validation performed.

Do not claim completion if validation fails.
