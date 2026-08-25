# First DATA ANALYSIS ASSESSMENT — one linear Online Retail notebook

This is a **cell plan**, not a completed notebook. Build one notebook from top to bottom. Enter and run one code cell at a time; do not jump ahead. Each copy-ready block is labelled **Markdown cell text** or **Code cell text** so you can place it directly into the matching notebook cell.

I have chosen the Online Retail Dataset - you may have chosen something different. The principles apply to all dataset choices. The Markdown cell text explains what you are doing and why; the Code cell text is the work you carry out and inspect.

The dataset mentioned here is `Data_Sets/Online_Retail.csv`because that is what I am using, and I wrote this for me.  However, the notebook should use relative paths only, so it remains portable.  It applies to your dataset if you use the same folder structure, and replace references to my dataset with your one.

This works on any machine provided the project structure and current working directory are correct - and all the files are in the same project folder.

If the project structure is nested:

"your_project_folder"/
├── Data_Sets/
│   └── Online_Retail.csv
└── jupyter_notebooks/
    └── your_notebook.ipynb
    
Because the notebook is inside the folder `jupyter_notebooks`, the path in the code goes up one level to the project folder before entering `Data_Sets`. The two dots in `../` mean “the parent folder”.

So the code looks like:

```python
dataset_path = Path("../Data_Sets/Online_Retail.csv")
df_raw = pd.read_csv(dataset_path)
```

However, if you keep a flat structure it is even easier:

"your_project_folder"/
    └── Online_Retail.csv
    └── your_notebook.ipynb
    
You can use:

```python
dataset_path = Path("Online_Retail.csv")
df_raw = pd.read_csv(dataset_path)
```

## The single notebook structure

This project should be simple - don't bother creating multiple notebooks as they will all simply repeat each other then add a bit more code on at the end.  Wasteful.  Stick to one notebook that flows from start to finish, and follows these phases that you can make clear in the markdown explanation cells.

1. **Setup and extraction** — import the tools and load the raw file.
2. **ETL and data quality** — inspect first, record decisions, clean, transform, and validate.
3. **Exploratory analysis** — define a question and metric before each result.
4. **Visualisation** — create static Matplotlib/Seaborn charts that answer the questions.
5. **Findings and limits** — record what the data supports, what it cannot prove, and what you would investigate next.

The notebook is deliberately linear. Every later cell uses objects created earlier; there are no three independent “starting” notebooks.

I have noted in steps what I need to explain in #markdown and below that I indicate what I need to run in the ``python`` code cell.

## Preferred analysis-note formatting

Use this simple style for my own interpretation after an output:

```markdown
**Analysis:**

This paragraph explains what the output appears to show.

**Important count:** 10,624

**Second count:** 1,336
```

Keep a blank line after `**Analysis:**` so the heading and the explanation do not run together. Use bold labels and short paragraphs to distinguish my reasoning from code output. Use colour only as an optional extra; the notebook should remain clear and readable in plain Markdown and on GitHub.

## Golden evidence rule

**Do not extrapolate beyond displayed evidence.**

Before stating a finding:

1. Write the code that reveals the relevant values, rows, counts, or categories.
2. Run the cell and inspect the visible output.
3. Describe only what that output demonstrates.
4. Mark anything not yet demonstrated as a question or hypothesis, not as a finding.

This rule applies to every ETL decision, table, visualisation, and conclusion. A hidden calculation, an unseen row, or prior knowledge of the dataset is not evidence until the notebook displays it.

## Assessment coverage map

The finished notebook must make the pass criteria visible through its Markdown, code, outputs, and reviewed findings. The overall project hypothesis and business case can be developed in the README; the notebook will carry the supporting questions, metrics, evidence, and limitations.

- **LO1.1:** introduce the main Online Retail analytics applications: growth-opportunity analysis, underperformance signals, product performance, sales trends, country comparisons, customer behaviour, and possible segmentation.
- **LO1.2:** state the retail-data challenge and a clearly labelled AI opportunity. The proposed AI opportunity is to prioritise markets or customer groups and flag unusual performance using the measured features; it is not claimed as a model produced by this descriptive notebook.
- **LO2.1:** use Python, Pandas, Matplotlib, Seaborn, and other relevant tools to manipulate, analyse, and visualise the data. Plotly remains optional; one Plotly chart must never replace the static charts needed for GitHub.
- **LO2.2:** include a short code-review note explaining corrections, improvements, and why the chosen techniques were appropriate.
- **LO2.3:** show the complete public-dataset workflow: cleaning, transformation, exploratory analysis, and visualisation.
- **LO3.1:** record how Codex or another generative AI tool assisted with code, investigation, or design, and how the suggestions were checked.
- **LO3.2:** record at least one example where generative AI helped turn visible output into a concise narrative, followed by the learner's evidence check and final wording.
- **LO3.3:** finish with findings, limitations, uncertainties, and reasonable alternatives.
- **LO4.1:** show collection, cleaning, processing, validation, and storage of the data.
- **LO4.2:** explain the rationale for the data-handling and processing choices through Markdown and code comments.

---

# Phase 1 — setup and extraction

**Markdown cell text:**

### Business intent — growth and performance

> This notebook is written for the Commercial Director of a UK-based online retailer. The client wants evidence to identify growth opportunities and signals of underperformance across markets, products, customers, and time periods.
>
> The analysis will not assume that the busiest market is the most valuable. It will compare transaction volume with revenue, average transaction value, customer activity, product range, and trends. One question is whether a market such as `Netherlands` could offer higher commercial value than transaction counts alone suggest.
>
> The overall hypothesis is documented in the supporting README notes. The notebook tests it through specific questions, metrics, tables, and visualisations. A future AI opportunity would be to prioritise markets or customer groups and flag unusual performance patterns; this descriptive notebook does not claim to build that AI system.

**Markdown cell text:**

### Cell 1 — import the toolkit

> Import the libraries used throughout the notebook. Pandas works with table-shaped data; NumPy supports numerical work; Matplotlib and Seaborn create the visualisations. `pd`, `np`, `plt`, and `sns` are short names used later.

**Code cell text:**

```python
from pathlib import Path

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_theme(style="whitegrid")
```

**What it does:** loads the tools and sets a readable default style. It does not load or change the dataset.

**Markdown cell text:**

### Cell 2 — locate and load the raw dataset

> Set the relative file path from the `jupyter_notebooks` folder and read the CSV into `df_raw`. The `_raw` suffix means this is the untouched source table. Keep it unchanged so that every cleaning decision can be checked against the original.

**Code cell text:**

```python
dataset_path = Path("../Data_Sets/Online_Retail.csv")
df_raw = pd.read_csv(dataset_path)

print("Loaded:", dataset_path)
print("Shape:", df_raw.shape)
```

**What to inspect:** confirm that the output shows the expected file and shape. Here, `../` moves from `jupyter_notebooks` up to the project folder.

**Markdown cell text:**

### Cell 3 — first look before changing anything

> Establish what one row represents and what kinds of fields the raw table contains. These checks are reconnaissance, not cleaning.

**Code cell text:**

```python
display(df_raw.head())
print("Columns:", df_raw.columns.tolist())
print("\nData types:")
display(df_raw.dtypes)
```

**What to write in your notes:** one row appears to represent a product line on an invoice. Confirm that interpretation from the column names rather than assuming it.

---

## Phase 2 — ETL and data quality

**Markdown cell text:**

### Cell 4 — make a quality inventory

> Count missing values, exact duplicate rows, non-positive quantities, and non-positive prices before removing anything. The counts are evidence for later decisions.

**Code cell text:**

```python
print("Missing values by column:")
display(df_raw.isna().sum())

print("Exact duplicate rows:", df_raw.duplicated().sum())
print("Rows with Quantity <= 0:", (df_raw["Quantity"] <= 0).sum())
print("Rows with UnitPrice <= 0:", (df_raw["UnitPrice"] <= 0).sum())
```

**What to record:** write the counts and a one-sentence question beside each suspicious field. Do not call a value “wrong” until you know what it means.

**Markdown cell text:**

### Cell 5 — inspect cancellations and non-product stock codes

> Negative quantities may represent cancellations rather than ordinary sales. Stock codes containing letters may be postage, fees, or other non-product rows. Inspect these groups before choosing a rule.

**Code cell text:**

```python
invoice_text = df_raw["InvoiceNo"].astype("string")
stock_text = df_raw["StockCode"].astype("string").str.strip()

print("Rows whose invoice number starts with C:", invoice_text.str.startswith("C").sum())
print("Negative-quantity rows with a C invoice:",
      ((df_raw["Quantity"] < 0) & invoice_text.str.startswith("C")).sum())

non_product_codes = df_raw.loc[~stock_text.str.match(r"^\d"), "StockCode"]
print("Most common non-numeric-leading stock codes:")
display(non_product_codes.value_counts().head(20))
```

**Markdown cell text:**

### Cell 5A — investigate negative quantities that are not C invoices

> Cell 5 gives us the initial counts. The next question follows directly from those counts: among the negative-quantity rows, which ones have the `C` invoice prefix and which ones do not? Cell 5A investigates the non-`C` group before we decide how to treat it. The group cannot yet be labelled as cancellations.

**Code cell text:**

```python
cancellation_mask = df_raw["InvoiceNo"].astype("string").str.startswith("C", na=False)
negative_quantity_mask = df_raw["Quantity"] < 0
other_negative_mask = negative_quantity_mask & ~cancellation_mask
other_negative_rows = df_raw.loc[other_negative_mask].copy()

print("Negative-quantity rows with a C invoice:",
      (negative_quantity_mask & cancellation_mask).sum())
print("Negative-quantity rows without a C invoice:",
      other_negative_mask.sum())
print("Missing descriptions in the non-C negative group:",
      other_negative_rows["Description"].isna().sum())
print("Present descriptions in the non-C negative group:",
      other_negative_rows["Description"].notna().sum())
print("Unit prices in the non-C negative group:")
display(other_negative_rows["UnitPrice"].value_counts(dropna=False).rename_axis("UnitPrice").to_frame("Rows"))

print("Description values present in the non-C negative group:")
description_counts = (
    other_negative_rows["Description"]
    .value_counts(dropna=False)
    .rename_axis("Description")
    .to_frame("Rows")
)
display(description_counts)

print("Rows with descriptions present:")
display(other_negative_rows.loc[
    other_negative_rows["Description"].notna(),
    ["InvoiceNo", "StockCode", "Description", "Quantity", "UnitPrice", "Country"]
].sort_values("Description"))
```

**What to record:** use the visible counts and description table to describe what the 1,336 exception rows represent. Do not remove them until their meaning has been considered.

**Markdown cell text:**

### Cell 5B — investigate non-product stock codes

> After Cell 5A, use the visible descriptions, prices, and rows to record what that exception group appears to contain. Cell 5B now follows the other question raised by Cell 5: do the non-numeric-leading stock codes represent ordinary products, or a separate set of operational or adjustment records? The code pattern identifies a group for investigation; it does not prove that every row should be removed.

**Code cell text:**

```python
stock_text = df_raw["StockCode"].astype("string").str.strip()
non_product_mask = ~stock_text.str.match(r"^\d", na=False)

non_product_summary = (
    df_raw.loc[non_product_mask]
      .groupby(["StockCode", "Description"], dropna=False)
      .agg(Rows=("StockCode", "size"),
           TotalQuantity=("Quantity", "sum"),
           LowestUnitPrice=("UnitPrice", "min"),
           HighestUnitPrice=("UnitPrice", "max"))
      .sort_values("Rows", ascending=False)
)

display(non_product_summary.head(20))
```

**What to record:** note which codes appear to be postage, charges, adjustments, or other operational records, and which descriptions remain unclear.

**Markdown cell text:**

**Analysis:**

Cell 5B shows that the non-numeric stock-code population is mixed. Postage groups dominate the flagged rows by count, but this summary does not show their effect on revenue or product trends. Other groups have product-like descriptions or different price and quantity patterns, so a blanket removal rule would be unsafe. Cell 5B1 will measure the complete population before we record the ETL decisions.

**Markdown cell text:**

### Cell 5B1 — audit the complete non-numeric stock-code population

> Cell 5B showed a mixed set of non-numeric stock codes, but only the 20 largest groups were visible. This cell expands the check to every non-numeric stock-code and description group, displaying counts, quantities, price ranges, zero- and negative-price counts, negative-quantity counts, missing descriptions, and relevant rows for follow-up. These results will support the category-by-category ETL decisions.

**Code cell text:**

```python
# Keep stock codes that do not begin with a digit.
stock_text = df_raw["StockCode"].astype("string").str.strip()
non_product_mask = ~stock_text.str.match(r"^\d", na=False)

non_product_rows = df_raw.loc[non_product_mask].copy()

# Create flags so each issue can be counted by stock-code group.
non_product_rows["ZeroPrice"] = non_product_rows["UnitPrice"] == 0
non_product_rows["NegativePrice"] = non_product_rows["UnitPrice"] < 0
non_product_rows["NegativeQuantity"] = non_product_rows["Quantity"] < 0
non_product_rows["MissingDescription"] = non_product_rows["Description"].isna()

# Summarise every non-numeric stock-code and description group.
non_product_summary = (
    non_product_rows
    .groupby(["StockCode", "Description"], dropna=False)
    .agg(
        Rows=("StockCode", "size"),
        TotalQuantity=("Quantity", "sum"),
        LowestUnitPrice=("UnitPrice", "min"),
        HighestUnitPrice=("UnitPrice", "max"),
        ZeroPriceRows=("ZeroPrice", "sum"),
        NegativePriceRows=("NegativePrice", "sum"),
        NegativeQuantityRows=("NegativeQuantity", "sum"),
        MissingDescriptionRows=("MissingDescription", "sum"),
    )
    .sort_values("Rows", ascending=False)
)

display(non_product_summary.reset_index())

# Display rows needing direct description or price follow-up.
follow_up_mask = (
    non_product_rows["MissingDescription"]
    | (non_product_rows["UnitPrice"] <= 0)
)

print("Rows with a missing description or non-positive price:",
      follow_up_mask.sum())

display(non_product_rows.loc[
    follow_up_mask,
    ["InvoiceNo", "StockCode", "Description",
     "Quantity", "UnitPrice", "Country"]
].sort_values(["StockCode", "Description"], na_position="first"))
```

**Markdown cell text:**

**Analysis:**

Cell 5B1 shows that the non-numeric stock-code population is mixed. Postage groups dominate the flagged rows by count, but the summary does not show their effect on revenue or product trends. Other groups have product-like descriptions or different price and quantity patterns, so a blanket removal rule would be unsafe. Cell 5B2 will measure the analytical impact of these groups before the ETL decisions are recorded.

**Markdown cell text:**

### Cell 5B2 — measure the analytical impact of the identified groups

> Cell 5B1 summarised and flagged the non-numeric stock-code groups, but counts alone do not show whether they materially affect the measures we plan to analyse. This cell compares the groups using row counts, total quantity, provisional line revenue (`Quantity × UnitPrice`), and their shares of the overall table. This provides impact evidence for the category-by-category ETL decisions.

**Code cell text:**

```python
# Calculate provisional line revenue without changing the raw table.
impact_rows = non_product_rows.copy()
impact_rows["LineRevenue"] = impact_rows["Quantity"] * impact_rows["UnitPrice"]

total_rows = len(df_raw)
total_line_revenue = (df_raw["Quantity"] * df_raw["UnitPrice"]).sum()

impact_summary = (
    impact_rows
    .groupby(["StockCode", "Description"], dropna=False)
    .agg(
        Rows=("StockCode", "size"),
        TotalQuantity=("Quantity", "sum"),
        TotalRevenue=("LineRevenue", "sum"),
    )
    .assign(
        ShareOfRows=lambda table: table["Rows"] / total_rows * 100,
        ShareOfRevenue=lambda table: table["TotalRevenue"] / total_line_revenue * 100,
    )
    .sort_values("TotalRevenue", ascending=False)
)

print("Non-numeric stock-code rows:", len(non_product_rows))
print(f"Share of all rows: {len(non_product_rows) / total_rows * 100:.2f}%")
print("Non-numeric stock-code line revenue:",
      f"£{impact_rows['LineRevenue'].sum():,.2f}")
print(f"Share of all line revenue: {impact_rows['LineRevenue'].sum() / total_line_revenue * 100:.2f}%")

display(impact_summary.reset_index())
```

**What to record:** record the visible row and revenue shares for the identified groups. Use these figures alongside the descriptions and price patterns; do not treat a small row count as proof that a group is irrelevant.

**Markdown cell text:**

### Cell 5C — investigate non-positive prices and missing descriptions

> Cells 5B1 and 5B2 show the complete groups and their provisional analytical impact. The next question is whether zero or negative prices, and missing descriptions, occur in these or other rows. Cell 5C separates those conditions so they are not collapsed into one automatic cleaning rule.

**Code cell text:**

```python
zero_price_mask = df_raw["UnitPrice"] == 0
negative_price_mask = df_raw["UnitPrice"] < 0
missing_description_mask = df_raw["Description"].isna()

print("Rows with UnitPrice == 0:", zero_price_mask.sum())
print("Rows with UnitPrice < 0:", negative_price_mask.sum())
print("Rows with missing Description:", missing_description_mask.sum())

suspicious_rows = df_raw.loc[
    zero_price_mask | negative_price_mask | missing_description_mask,
    ["InvoiceNo", "StockCode", "Description", "Quantity", "UnitPrice"]
].sort_values(["StockCode", "Description"], na_position="first")

print("Rows requiring direct inspection:", len(suspicious_rows))
display(suspicious_rows)
```

**Markdown cell text:**

> Cells 5A–5C now provide the evidence for the ETL decision note. Write only what the visible outputs show; keep any unresolved meaning as a question or hypothesis rather than presenting it as a fact.

### Decision checkpoint — record the ETL decisions

Before the next code cell, write a short Markdown note stating what you will do with:

- exact duplicates;
- C-invoice cancellations;
- the 1,336 negative-quantity exceptions;
- non-product stock codes;
- zero and negative prices; and
- missing descriptions.

Each decision should name the evidence and say whether the rows are removed, retained, or treated separately. Do not run the cleaning cell until this note is written.

**Markdown cell text:**

### Cell 6 — create a working copy and standardise fields

> Now create the working copy and standardise text and date fields. This is a reversible transformation step; it does not remove rows or replace the decisions you have just documented.

**Code cell text:**

```python
df = df_raw.copy()

df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"])
df["InvoiceNo"] = df["InvoiceNo"].astype("string").str.strip()
df["StockCode"] = df["StockCode"].astype("string").str.strip()
df["Description"] = df["Description"].astype("string").str.strip()
```

**What it does:** makes dates usable for time analysis and removes accidental surrounding spaces from text fields. It does not remove rows.

**Markdown cell text:**

### Cell 7 — apply the confirmed cleaning rules

> Apply only the rules justified in the decision note above. This cell is deliberately after the investigations, so the filters are visible consequences of evidence rather than assumptions.

**Code cell text:**

```python
duplicate_mask = df.duplicated()
cancellation_mask = df["InvoiceNo"].str.startswith("C", na=False)
non_product_mask = ~df["StockCode"].str.match(r"^\d", na=False)
non_positive_price_mask = df["UnitPrice"] <= 0

df_clean = df.loc[
    ~duplicate_mask
    & ~cancellation_mask
    & ~non_product_mask
    & ~non_positive_price_mask
].copy()
```

**Important:** this example leaves the non-C negative-quantity exceptions and missing descriptions in `df_clean` until your written decisions say otherwise. Add or remove filters only after documenting the reason.

**Markdown cell text:**

### Cell 8 — create analysis fields

> Add fields that express the business measures needed later. `Revenue` is line revenue, calculated as quantity multiplied by unit price. The time fields support monthly and weekday comparisons.

**Code cell text:**

```python
df_clean["Revenue"] = df_clean["Quantity"] * df_clean["UnitPrice"]
df_clean["Month"] = df_clean["InvoiceDate"].dt.to_period("M").astype("string")
df_clean["DayOfWeek"] = df_clean["InvoiceDate"].dt.day_name()
```

**What to inspect:** display `df_clean[["Quantity", "UnitPrice", "Revenue", "Month"]].head()` and check that the values make sense.

**Markdown cell text:**

### Cell 9 — validate and save the clean table

> Check that the cleaning rules produced the intended state, then save a named clean file. This creates a reproducible hand-off between ETL and analysis without requiring a second notebook.

**Code cell text:**

```python
print("Clean shape:", df_clean.shape)
print("Remaining duplicates:", df_clean.duplicated().sum())
print("Remaining non-positive quantities:", (df_clean["Quantity"] <= 0).sum())
print("Remaining non-positive prices:", (df_clean["UnitPrice"] <= 0).sum())
print("Remaining missing values:")
display(df_clean.isna().sum())

clean_path = Path("../outputs/cleaned_online_retail.csv")
clean_path.parent.mkdir(exist_ok=True)
df_clean.to_csv(clean_path, index=False)
print("Saved:", clean_path)
```

**Checkpoint:** do not continue until the validation output matches the decisions you recorded.

**Markdown cell text:**

### Code review — evaluate the workflow

> Review the completed ETL section before beginning analysis. Record at least one correction or improvement, explain why the change was made, and identify one alternative that could have been used. For example, explain why the raw table was preserved, why the investigations were placed before cleaning, and why the final fields were created only after the cleaning decisions.

---

## Phase 3 — exploratory analysis

For every analysis cell, write the human question and metric in the Markdown above it. A chart is not a question by itself.

**Markdown cell text:**

### Cell 10 — headline metrics

> Summarise the cleaned sales table before choosing detailed charts. Report the row count, invoice count, customer count, product count, date range, and total line revenue.

**Code cell text:**

```python
print("Rows:", len(df_clean))
print("Invoices:", df_clean["InvoiceNo"].nunique())
print("Customers:", df_clean["CustomerID"].nunique())
print("Products:", df_clean["StockCode"].nunique())
print("Date range:", df_clean["InvoiceDate"].min(), "to", df_clean["InvoiceDate"].max())
print(f"Line revenue: £{df_clean['Revenue'].sum():,.2f}")
```

**Markdown cell text:**

### Cell 11 — product performance table

> Question: which products contribute the most line revenue, and do they also sell the most units? Build a table first; the visual will come next.

**Code cell text:**

```python
product_summary = (
    df_clean.groupby(["StockCode", "Description"], dropna=False)
      .agg(TotalRevenue=("Revenue", "sum"),
           UnitsSold=("Quantity", "sum"),
           Invoices=("InvoiceNo", "nunique"))
      .sort_values("TotalRevenue", ascending=False)
)
display(product_summary.head(10))
```

**Markdown cell text:**

### Cell 12 — visualisation: top products by revenue

> Visual question: what are the ten highest-revenue products? A horizontal bar chart makes long product names readable. The metric is summed line revenue, not row count.

**Code cell text:**

```python
top_products = product_summary.head(10).sort_values("TotalRevenue")

fig, ax = plt.subplots(figsize=(10, 6))
sns.barplot(data=top_products.reset_index(), x="TotalRevenue", y="Description", ax=ax)
ax.set_title("Top 10 products by line revenue")
ax.set_xlabel("Revenue (£)")
ax.set_ylabel("Product")
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 13 — country metrics and visualisation

> Question: do the countries with the most revenue also have the most transaction volume? Compare two different metrics so value is not confused with activity.

**Code cell text:**

```python
country_summary = (
    df_clean.groupby("Country")
      .agg(TotalRevenue=("Revenue", "sum"),
           Transactions=("InvoiceNo", "nunique"),
           UnitsSold=("Quantity", "sum"))
      .sort_values("TotalRevenue", ascending=False)
)

plot_countries = country_summary.head(10).sort_values("TotalRevenue")
fig, axes = plt.subplots(1, 2, figsize=(14, 6))
sns.barplot(data=plot_countries.reset_index(), x="TotalRevenue", y="Country", ax=axes[0])
sns.barplot(data=plot_countries.reset_index(), x="Transactions", y="Country", ax=axes[1])
axes[0].set_title("Top countries by revenue")
axes[1].set_title("The same countries by transactions")
for ax in axes:
    ax.set_ylabel("")
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 14 — monthly revenue trend

> Question: how does recorded revenue change over time? Aggregate line revenue by month and show the time order explicitly. Note that the final month is partial if the source stops early in that month.

**Code cell text:**

```python
monthly_revenue = (
    df_clean.groupby("Month", as_index=False)["Revenue"]
      .sum()
      .sort_values("Month")
)

fig, ax = plt.subplots(figsize=(12, 5))
sns.lineplot(data=monthly_revenue, x="Month", y="Revenue", marker="o", ax=ax)
ax.set_title("Monthly line revenue")
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (£)")
ax.tick_params(axis="x", rotation=45)
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 15 — revenue distribution

> Question: are most line revenues small, with a few unusually large values? A distribution chart helps decide whether totals are being driven by a long tail. Use a log-scaled x-axis only to make the shape readable; explain that choice.

**Code cell text:**

```python
fig, ax = plt.subplots(figsize=(10, 5))
sns.histplot(data=df_clean, x="Revenue", bins=60, log_scale=True, ax=ax)
ax.set_title("Distribution of line revenue")
ax.set_xlabel("Line revenue (£, logarithmic scale)")
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 16 — seasonality heatmap

> Question: does recorded revenue vary by month and day of week? A heatmap is appropriate because two categorical time dimensions form a grid.

**Code cell text:**

```python
weekday_order = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
seasonality = df_clean.pivot_table(
    index="DayOfWeek", columns="Month", values="Revenue", aggfunc="sum"
).reindex(weekday_order)

fig, ax = plt.subplots(figsize=(14, 5))
sns.heatmap(seasonality, cmap="YlGnBu", ax=ax)
ax.set_title("Revenue by weekday and month")
ax.set_xlabel("Month")
ax.set_ylabel("Day of week")
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 17 — customer-level RFM preparation

> Question: how do customer recency, purchase frequency, and monetary value vary? Build the three measures before plotting them. Recency is days since a customer’s latest invoice; frequency is distinct invoices; monetary value is summed revenue.

**Code cell text:**

```python
analysis_date = df_clean["InvoiceDate"].max()

rfm = (
    df_clean.groupby("CustomerID")
      .agg(Recency=("InvoiceDate", lambda dates: (analysis_date - dates.max()).days),
           Frequency=("InvoiceNo", "nunique"),
           Monetary=("Revenue", "sum"))
      .reset_index()
)
display(rfm.head())
```

**Decision checkpoint:** the current source uses `CustomerID == 15287` as a placeholder for unattributed customers. Investigate and document that issue before deciding whether to exclude it from RFM. Do not silently treat the placeholder as a real person.

**Markdown cell text:**

### Cell 18 — visualisation: frequency versus monetary value

> Visual question: is monetary value concentrated among customers with many invoices? Use a scatter plot and a logarithmic scale when the values are strongly skewed. If you exclude a documented placeholder, say so in this Markdown cell and in the findings.

**Code cell text:**

```python
fig, ax = plt.subplots(figsize=(10, 7))
sns.scatterplot(data=rfm, x="Frequency", y="Monetary", alpha=0.5, ax=ax)
ax.set_xscale("log")
ax.set_yscale("log")
ax.set_title("Customer frequency versus monetary value")
ax.set_xlabel("Distinct invoices (log scale)")
ax.set_ylabel("Revenue (£, log scale)")
plt.tight_layout()
plt.show()
```

**Markdown cell text:**

### Cell 19 — one evidence-led branch

> Choose one unexpected result from the earlier tables or charts and investigate it. State the question, the metric, and why this branch follows from the evidence. Do not add a chart simply because it is available.

**Code cell text (choose your branch and write it):**

```python
# Your own follow-up calculation and visualisation go here.
# Start by writing the question in the Markdown cell above.
```

---

## Phase 4 — findings and reflection

**Markdown cell text:**

### AI assistance and evidence review

> Record one or two examples of how Codex or another generative AI tool assisted this project. State the task, what the tool suggested, what you checked in the visible notebook output, and what you accepted, changed, or rejected. This documents the AI-assisted workflow without presenting AI as the source of the evidence.

**Markdown cell text:**

### AI-assisted storytelling

> I consulted Codex while interpreting selected tables and visualisations. Codex helped me turn visible counts, rows, and chart results into concise narrative summaries. I checked each suggested statement against the displayed evidence, removed unsupported claims, and retained only conclusions justified by the notebook output. The final findings are my reviewed interpretations, not unverified AI-generated conclusions.

This final section is Markdown only. Write short, evidence-based paragraphs under these headings:

1. **What the raw data contained** — include the important quality counts.
2. **What each cleaning decision changed** — state what was removed or retained and why.
3. **What the visualisations showed** — one finding per chart, tied to its metric.
4. **What remains uncertain** — especially partial-period coverage, placeholder customer IDs, and the absence of cost or margin data.
5. **Next question** — one follow-up investigation justified by something you observed.

Do not claim that a descriptive chart proves cause, intent, profitability, or customer identity.

## Final run checklist

- Restart the kernel and run every cell from the top.
- Confirm no cell depends on an object created later.
- Confirm the clean file is written to `../outputs/cleaned_online_retail.csv` from the notebook's nested location.
- Confirm the notebook contains the visualisations, not just calculations.
- Confirm every chart has a question, metric, title, axis labels, and a short interpretation.
- Confirm the opening context covers the Online Retail analytics applications, the retail challenge, and the proposed AI opportunity.
- Confirm the code-review note and AI-assistance record are present and distinguish suggestions from verified evidence.
- Confirm the final findings include an AI-assisted narrative that was checked against visible output.
- Keep Plotly optional; if one interactive chart is included, retain a static Matplotlib or Seaborn equivalent for GitHub.
- Keep this plan beside the notebook as your build map; the finished notebook is the executed evidence.
