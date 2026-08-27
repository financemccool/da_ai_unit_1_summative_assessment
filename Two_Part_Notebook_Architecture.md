# Two-part notebook architecture

## Purpose

This document records the reasoning behind the project's two-notebook structure. It is an architectural guide, not a second cell-by-cell build plan. The detailed sequence remains in `Notebook_Cell_Plan.md`.

The project is deliberately split into two linear notebooks because the dataset evaluation and cleaning work became a substantial investigation in its own right. The business analysis should not begin until that investigation has established what the data can reliably support.

> **A chart is not a question by itself.** Every visual should answer a human question using a defined metric, an appropriate data shape, and an explanation of what the result means.

## The whole route

```text
Raw CSV
  ↓
Part 1 — evaluate, clean, standardise, validate
  ↓
Validated paid-product hand-off
  ↓
Part 2 — investigate, visualise, interpret
  ↓
Findings, limitations, and next questions
```

The notebooks are run in order. Part 1 protects the raw evidence and produces the single validated table that Part 2 consumes. Part 2 does not repeat the cleaning decisions or quietly rebuild the dataset.

## Why the project is split

The original design treated the work as one notebook. That became difficult to read and difficult to audit because three different kinds of reasoning were becoming mixed together:

1. What is present in the supplied data?
2. Which rows and fields can support a defensible paid-product analysis?
3. What business patterns are visible after that decision?

The split gives each notebook one clear responsibility. It also makes the transition between data preparation and analysis visible, reproducible, and testable.

## Part 1 — evaluation, cleaning, and standardisation

**Notebook:** `2_part_notebook/01_evaluation_clean_standardise.ipynb`

### Human question

What does the supplied dataset contain, which quality signals matter, and what cleaned scope can I defend before answering business questions?

### Route

```text
Setup and extraction
  → first inspection of the untouched raw table
  → data-quality investigation
  → decision checkpoint
  → standardise fields in a working copy
  → apply confirmed exclusion and cancellation rules
  → create business measures and time fields
  → validate before and after
  → save the Part 2 hand-off
```

### What Part 1 establishes

- the row grain, columns, types, and date coverage of the raw table;
- the evidence behind the treatment of cancellation-style invoices, negative quantities, non-product lines, missing descriptions, prices, and duplicates;
- the investigation of the apparent CustomerID outlier and its data-version explanation;
- a clean paid-product scope with the exclusion decisions kept visible;
- `Revenue`, `Month`, and `DayOfWeek` fields for the later analysis;
- a before-and-after validation comparison;
- a saved, reproducible hand-off for Part 2.

The raw table remains available as an audit source. Standardisation and cleaning operate on a working copy, so the decisions remain reversible and inspectable.

## Part 2 — investigation and visualisation

**Notebook:** `2_part_notebook/02_investigate_visualise.ipynb`

### Human question

What product, market, customer-concentration, and time patterns are visible in the validated paid-product table, and what might they mean for the imagined Commercial Director?

### Route

```text
Reload the Part 1 hand-off
  → verify its shape and required fields
  → investigate product and market patterns
  → examine transaction value and customer concentration
  → investigate monthly, weekly, and seasonal movement
  → compare domestic and international contribution
  → interpret findings with limitations
  → record next questions
```

Part 2 is the analytical and visual layer. It follows the question-first method: state the human question and metric, inspect the relevant table or grouped shape, choose the chart that makes that comparison readable, and explain the visible result.

The notebook contains the complete visual record for the business questions. The README shows only a curated selection of those charts.

## The handoff contract

Part 1 writes:

```text
outputs/cleaned_online_retail.csv
```

Part 2 reloads that file before doing business analysis. The hand-off contains the paid-product working table and the analysis fields created by Part 1, including:

```text
InvoiceNo, StockCode, Description, Quantity, InvoiceDate,
UnitPrice, CustomerID, Country, Revenue, Month, DayOfWeek
```

The hand-off checkpoint confirms that the file exists, can be loaded, and contains the fields required by the later questions. This is the boundary between data preparation and interpretation.

## What belongs in each notebook

| Responsibility | Part 1 | Part 2 |
|---|---:|---:|
| Preserve and inspect the raw source | Yes | No |
| Investigate data-quality signals | Yes | No |
| Decide and document the analytical scope | Yes | No |
| Standardise and clean the working table | Yes | No |
| Validate and save the hand-off | Yes | No |
| Reload the validated hand-off | No | Yes |
| Answer focused business questions | No | Yes |
| Build the business visualisations | No | Yes |
| Interpret findings and limitations | No | Yes |

This boundary prevents later charts from hiding unresolved data decisions and prevents Part 1 from becoming overloaded with conclusions that depend on the cleaned table.

## Relationship to the detailed cell plan

`Notebook_Cell_Plan.md` records the detailed construction history for the earlier single-notebook route. This document records the current two-part architecture and the value that travels between the parts. It should be read first when someone needs the project map; the detailed plan is useful when someone needs to trace individual cells and their teaching purpose.

