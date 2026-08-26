# Online Retail Analysis — simple notebook cell plan

**Status:** current simplified plan for the finished notebook. The earlier complex plan is retained only as a preserved reference.

I am keeping the existing complex notebook as a preserved reference. I had to pivot my plan when I discovered that it was built on stale information - and that the Kaggle dataset had been partially cleaned to remove missing Customer IDs - so my perception that there was a mystery Whale of a customer to investigate evaporated in a category tidy up.  That plan was complex and nuanced and too much for the task now in hand.  This version reduces the amount of code I need to hold in my head while keeping the investigation, the decision checkpoint, the assessment evidence, and my own Markdown voice.

## My design decisions

- I use one linear notebook and run it from top to bottom.
- I use one dataset: `Data_Sets/Online_Retail.csv`.
- I keep `df_raw` as the untouched audit source, `df` as the working copy, `df_clean` as the paid-product analysis table, and `df_excluded` as the audit view of excluded rows.
- I keep the NumPy import because it is part of my normal toolkit, even if this analysis does not need a separate NumPy calculation.
- I use Matplotlib for one clear before-and-after ETL comparison.
- I use Seaborn for the later analytical charts.
- I do not use Plotly. Static charts are sufficient for this assessment and are easier to reproduce in the notebook and on GitHub.
- I keep the investigation before the cleaning decision. I do not silently remove rows before I have recorded what they represent.
- Every learner-facing Markdown cell is written in the first person. I preserve my personal reactions, decision trail, “Happy Coder” sign-off, AI disclosure, and limitations where they still fit the shorter flow.
- Every code cell contains short `#` comments where the purpose of a step, mask, rule, or calculation needs a reminder. The comments explain what I am doing and why; they do not replace the Markdown explanation.
- I apply the golden evidence rule: I display the values, rows, counts, or categories before I write a finding about them.

## The simple notebook shape

The notebook has five parts:

1. **Set up and inspect** — load the raw table and establish its shape.
2. **Investigate data quality** — display the suspicious groups before deciding what to do.
3. **Clean and validate** — apply the recorded rules, create analysis fields, and compare before with after.
4. **Answer three focused questions** — product revenue, country value versus activity, and monthly recorded revenue.
5. **Write the findings** — record evidence, limitations, AI assistance, code review, and next questions.

The plan counts **code cells only**. Markdown cells sit above the related code cell. The decision checkpoint and final findings are Markdown-only cells and are not given a code-cell number.

## Part 1 — set up and inspect

### Markdown before code cell 1 — business intent

> I write this notebook for the Commercial Director of a UK-based online retailer. I am looking for evidence of product performance, market value, activity over time, and possible underperformance. I will compare transaction volume with revenue rather than assuming that the busiest market is the most valuable.

> The wider retail questions include growth opportunities, product performance, sales trends, customer behaviour, market comparison, and possible future segmentation. This notebook makes a deliberately focused descriptive analysis. A future AI opportunity would be to prioritise markets or customer groups and flag unusual performance patterns from measured features; this notebook does not claim to build that AI system.

### Markdown before code cell 1 — import the toolkit

> I load the libraries used for the analysis and give them familiar short names. Pandas works with table-shaped data; NumPy remains part of my normal toolkit; Matplotlib and Seaborn create the visualisations.

> I am not using Plotly here. Static Matplotlib and Seaborn charts are more than sufficient for this assessment and for this dataset. Interactive charting would not add extra analytical value to the questions I am answering, so I am keeping the visual work static, focused, and easy to reproduce.

### Code cell 1 — import the toolkit

```python
# I load the libraries used for the analysis and give them familiar short names.
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# I use a readable default style for the later Seaborn charts.
sns.set_theme(style="whitegrid")
```

### Markdown before code cell 2 — load the raw file

> I load the supplied CSV into `df_raw`. I keep this first table unchanged so it remains my audit source throughout the notebook.

### Code cell 2 — load the raw data

```python
# I load the supplied CSV into an untouched raw table so later transformations remain reversible.
df_raw = pd.read_csv("../Data_Sets/Online_Retail.csv")
df_raw.head()
```

### Markdown before code cell 3 — establish the starting point

> I inspect the first rows, shape, column names, and data types before I make any changes. This gives me a visible starting point for the later cleaning comparison.

### Code cell 3 — first inspection

```python
# I establish the starting shape, fields, and data types before changing anything.
print("Shape:", df_raw.shape)
print("Columns:", list(df_raw.columns))
display(df_raw.head())
df_raw.info()
```

## Part 2 — investigate data quality before deciding

### Markdown before code cell 4 — quality inventory

> I count missing values, duplicate rows, non-positive quantities, non-positive prices, and cancellation-style invoice numbers. These are signals to investigate, not automatic instructions to delete rows.

### Code cell 4 — initial quality counts

```python
# I treat invoice numbers as text so I can inspect the cancellation-style C prefix.
invoice_text = df_raw["InvoiceNo"].astype("string")

# These counts describe quality signals; they do not decide the cleaning rules by themselves.
print("Missing values:")
display(df_raw.isna().sum())
print("Exact duplicate rows:", df_raw.duplicated().sum())
print("Quantity <= 0:", (df_raw["Quantity"] <= 0).sum())
print("UnitPrice <= 0:", (df_raw["UnitPrice"] <= 0).sum())
print("Invoice numbers starting with C:",
      invoice_text.str.startswith("C", na=False).sum())
```

### Markdown before code cell 5 — inspect cancellations and negative quantities

> Negative quantities may represent cancellations, returns, or another adjustment. I compare the `C`-invoice group with negative quantities that do not have the `C` prefix. I keep the meaning of the second group open until the displayed rows and descriptions give me enough evidence.

### Code cell 5 — cancellation evidence

```python
# I identify cancellation-style invoices and negative quantities separately.
cancellation_mask = invoice_text.str.startswith("C", na=False)
negative_quantity_mask = df_raw["Quantity"] < 0

# This mask isolates negative quantities that do not have the C invoice prefix.
other_negative_mask = negative_quantity_mask & ~cancellation_mask

print("Negative quantities with a C invoice:",
      (negative_quantity_mask & cancellation_mask).sum())
print("Negative quantities without a C invoice:",
      other_negative_mask.sum())

# I display sample exception rows so I can investigate their meaning before cleaning.
display(df_raw.loc[
    other_negative_mask,
    ["InvoiceNo", "StockCode", "Description", "Quantity", "UnitPrice", "Country"]
].head(20))
```

### Markdown before code cell 6 — inspect non-product stock codes

> Some stock codes do not begin with a digit. I inspect their descriptions and frequencies because the group can contain postage, fees, adjustments, vouchers, or product-like rows. The pattern identifies a group for investigation; it does not prove that every row should be removed.

### Code cell 6 — non-product evidence

```python
# I standardise stock codes as trimmed text before testing their first character.
stock_text = df_raw["StockCode"].astype("string").str.strip()

# This mask identifies stock codes that do not begin with a digit for investigation.
non_numeric_mask = ~stock_text.str.match(r"^\d", na=False)

# I inspect both the frequency of each code and a sample of the related rows.
non_product_rows = df_raw.loc[non_numeric_mask]
print("Rows with non-numeric-leading stock codes:", len(non_product_rows))
display(non_product_rows["StockCode"].value_counts().head(20))
display(non_product_rows[
    ["StockCode", "Description", "Quantity", "UnitPrice"]
].head(20))
```

### Markdown before code cell 7 — inspect prices, descriptions, and duplicates

> I separate zero prices, negative prices, missing descriptions, and duplicates. Keeping these conditions visible helps me explain the cleaning rules rather than hiding them inside a large unexplained mask.

### Code cell 7 — direct inspection of remaining quality signals

```python
# I keep each condition as a named mask so I can count and inspect it separately.
zero_price_mask = df_raw["UnitPrice"] == 0
negative_price_mask = df_raw["UnitPrice"] < 0
missing_description_mask = df_raw["Description"].isna()

print("UnitPrice == 0:", zero_price_mask.sum())
print("UnitPrice < 0:", negative_price_mask.sum())
print("Missing Description:", missing_description_mask.sum())

# I combine the visible conditions only for a focused inspection table.
display(df_raw.loc[
    zero_price_mask | negative_price_mask | missing_description_mask,
    ["InvoiceNo", "StockCode", "Description", "Quantity", "UnitPrice"]
].head(30))
```

### Markdown-only decision checkpoint

> Before I clean the data, I write a short decision note in my own words. The note records what I will do with exact duplicates, `C`-invoice cancellations, non-`C` negative quantities, non-product stock codes, non-positive prices, and missing descriptions.

> I name the evidence that supports each decision. I distinguish rows removed from the paid-product table, rows retained for audit, and product-like non-numeric rows retained because a blanket removal would be unsafe. This note is the point where investigation becomes a reproducible ETL rule.

## Part 3 — clean, transform, and validate

### Markdown before code cell 8 — create a working copy

> I now create a reversible working copy and standardise the fields needed later. This step does not remove rows or replace the decisions I have just documented.

### Code cell 8 — standardise fields

```python
# I copy the raw table so standardisation does not alter my audit source.
df = df_raw.copy()

# I convert dates and trim text fields for reliable later comparisons.
df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"])
df["InvoiceNo"] = df["InvoiceNo"].astype("string").str.strip()
df["StockCode"] = df["StockCode"].astype("string").str.strip()
df["Description"] = df["Description"].astype("string").str.strip()
```

### Markdown before code cell 9 — apply the confirmed rules

> I apply the rules from the decision checkpoint explicitly. The raw table remains available, and I preserve excluded rows in `df_excluded`. The clean table is my paid-product analysis view; it is not a claim that excluded operational or adjustment rows are meaningless.

### Code cell 9 — create the clean and excluded views

```python
# I create named masks so each confirmed cleaning rule remains visible.
duplicate_mask = df.duplicated()
cancellation_mask = df["InvoiceNo"].str.startswith("C", na=False)
negative_quantity_mask = df["Quantity"] < 0
non_c_negative_mask = negative_quantity_mask & ~cancellation_mask
non_positive_price_mask = df["UnitPrice"] <= 0
non_numeric_mask = ~df["StockCode"].str.match(r"^\d", na=False)

operational_codes = {
    "POST", "DOT", "M", "m", "C2", "D", "S",
    "BANK CHARGES", "AMAZONFEE", "CRUK", "B"
}
operational_mask = non_numeric_mask & (
    df["StockCode"].isin(operational_codes)
    | df["StockCode"].str.startswith("gift_", na=False)
)

# These codes were identified during the earlier investigation as operational,
# financial, adjustment, or voucher records rather than ordinary merchandise.
# Product-like non-numeric rows remain eligible unless the decision note says otherwise.
# I preserve every excluded row in a separate audit view.
df_excluded = df.loc[
    duplicate_mask
    | cancellation_mask
    | non_c_negative_mask
    | operational_mask
    | non_positive_price_mask
].copy()

df_clean = df.loc[
    ~duplicate_mask
    & ~cancellation_mask
    & ~non_c_negative_mask
    & ~operational_mask
    & ~non_positive_price_mask
].copy()

print("Clean shape:", df_clean.shape)
print("Excluded rows retained for audit:", df_excluded.shape[0])
```

### Markdown before code cell 10 — create the business measures

> I add the fields needed for the three questions. `Revenue` is line revenue: quantity multiplied by unit price. `Month` supports the time question, and `DayOfWeek` remains available for a later extension.

### Code cell 10 — create analysis fields

```python
# I calculate line revenue without changing the raw table.
df_clean["Revenue"] = df_clean["Quantity"] * df_clean["UnitPrice"]

# I create simple time fields for the monthly question and a possible weekday extension.
df_clean["Month"] = df_clean["InvoiceDate"].dt.to_period("M").astype("string")
df_clean["DayOfWeek"] = df_clean["InvoiceDate"].dt.day_name()

display(df_clean[["Quantity", "UnitPrice", "Revenue", "Month"]].head())
```

### Markdown before code cell 11 — validate and compare before with after

> I check that the clean table meets the rules I recorded. I then use one Matplotlib chart to compare the same quality conditions before and after ETL. The bars show condition counts, not unique excluded rows, because one row can satisfy more than one condition.

### Code cell 11 — validation and Matplotlib comparison

```python
# I recheck the same rules after cleaning so the visible result can be compared with my decisions.
print("Remaining duplicate rows:", df_clean.duplicated().sum())
print("Remaining cancellations:",
      df_clean["InvoiceNo"].str.startswith("C", na=False).sum())
print("Remaining Quantity <= 0:", (df_clean["Quantity"] <= 0).sum())
print("Remaining UnitPrice <= 0:", (df_clean["UnitPrice"] <= 0).sum())
print("Remaining missing values:")
display(df_clean.isna().sum())

# I place the before-and-after counts in one table so Matplotlib can compare them consistently.
comparison = pd.DataFrame({
    "Before ETL": [
        df_raw.duplicated().sum(),
        (df_raw["Quantity"] <= 0).sum(),
        (df_raw["UnitPrice"] <= 0).sum(),
        df_raw["Description"].isna().sum(),
    ],
    "After ETL": [
        df_clean.duplicated().sum(),
        (df_clean["Quantity"] <= 0).sum(),
        (df_clean["UnitPrice"] <= 0).sum(),
        df_clean["Description"].isna().sum(),
    ]
}, index=[
    "Exact duplicate rows",
    "Quantity <= 0",
    "UnitPrice <= 0",
    "Missing Description",
])

# I use one simple Matplotlib chart for the raw-versus-clean comparison.
comparison.plot(kind="bar", figsize=(10, 5), rot=0)
plt.title("Selected data-quality conditions before and after ETL")
plt.ylabel("Rows")
plt.tight_layout()
plt.show()

# I save the validated clean table as the hand-off for the analysis section.
df_clean.to_csv("../outputs/cleaned_online_retail.csv", index=False)
print("Saved cleaned table to ../outputs/cleaned_online_retail.csv")
```

## Part 4 — answer three focused business questions

### Markdown before code cell 12 — Question 1: which products generate recorded revenue?

> I start with product performance. I calculate recorded revenue by product and display the highest-revenue rows before I interpret the chart. This answers a product-performance question without claiming profit or margin.

### Code cell 12 — product revenue table

```python
# I group line revenue by product and keep the ten largest totals for the first question.
product_revenue = (
    df_clean.groupby(["StockCode", "Description"])["Revenue"]
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .reset_index()
)

display(product_revenue)
```

### Markdown before code cell 13 — product chart

> I use Seaborn to make the product comparison easier to read. I describe only the products and revenue values visible in the table and chart.

### Code cell 13 — Seaborn product chart

```python
# I use Seaborn to compare the products shown in the table.
plt.figure(figsize=(10, 6))
sns.barplot(data=product_revenue, x="Revenue", y="Description", color="steelblue")
plt.title("Top 10 products by recorded revenue")
plt.xlabel("Recorded revenue")
plt.ylabel("Product description")
plt.tight_layout()
plt.show()
```

### Markdown before code cell 14 — Question 1A: which products lead outside the UK?

> The full product chart is useful for total business scale, but United Kingdom purchases dominate the table. I create a second product view excluding the UK so international product demand is visible rather than flattened by the domestic market.

> This is a comparison choice, not a cleaning decision. The UK rows remain in `df_clean` and remain part of the overall analysis.

### Code cell 14 — non-UK product revenue chart

```python
# I create an international comparison view without changing the clean table.
df_non_uk = df_clean[df_clean["Country"] != "United Kingdom"].copy()

# I calculate the highest recorded-revenue products outside the UK.
non_uk_product_revenue = (
    df_non_uk.groupby(["StockCode", "Description"])["Revenue"]
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .reset_index()
)

display(non_uk_product_revenue)

# I use Seaborn to show international product demand at a readable scale.
plt.figure(figsize=(10, 6))
sns.barplot(
    data=non_uk_product_revenue,
    x="Revenue",
    y="Description",
    color="cornflowerblue",
)
plt.title("Top 10 products by recorded revenue outside the UK")
plt.xlabel("Recorded revenue outside the UK")
plt.ylabel("Product description")
plt.tight_layout()
plt.show()
```

### Markdown before code cell 15 — Question 1B: which international markets are buying those products?

> The non-UK product chart shows the international leaders, but it does not show which countries are buying them. I keep the top ten products and add country colour. I show the five largest non-UK markets individually and group the remaining countries as `Other international markets` so the legend remains readable.

### Code cell 15 — non-UK product and country hue chart

```python
# I keep the top ten international products from the previous comparison.
top_product_descriptions = non_uk_product_revenue["Description"].tolist()

# I identify the five largest non-UK markets for the colour groups.
top_non_uk_countries = (
    df_non_uk.groupby("Country")["Revenue"]
    .sum()
    .sort_values(ascending=False)
    .head(5)
    .index
    .tolist()
)

# I calculate product revenue by country for the selected products.
product_country_revenue = (
    df_non_uk[df_non_uk["Description"].isin(top_product_descriptions)]
    .groupby(["Description", "Country"])["Revenue"]
    .sum()
    .reset_index()
)

# I combine smaller markets so the hue legend remains readable.
product_country_revenue["CountryGroup"] = product_country_revenue["Country"].where(
    product_country_revenue["Country"].isin(top_non_uk_countries),
    "Other international markets",
)
product_country_revenue = (
    product_country_revenue
    .groupby(["Description", "CountryGroup"], as_index=False)["Revenue"]
    .sum()
)

display(product_country_revenue.sort_values("Revenue", ascending=False).head(20))

# I preserve the overall product ranking on the vertical axis.
product_order = non_uk_product_revenue["Description"].tolist()
plt.figure(figsize=(14, 8))
sns.barplot(
    data=product_country_revenue,
    x="Revenue",
    y="Description",
    hue="CountryGroup",
    order=product_order,
    palette="tab10",
)
plt.title("Who is buying the top products outside the UK?")
plt.xlabel("Recorded revenue outside the UK")
plt.ylabel("Product description")
plt.legend(title="Country group", bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout()
plt.show()
```

### Markdown-only analysis after code cell 15 — what does this product-by-country view tell me?

> This chart turns my top-product list into a market-mix question. I do not read `Other international markets` as one country: it combines the smaller international countries so that the main colour groups remain readable.

> The Netherlands is the clearest repeated pattern. It is the largest named market for `RABBIT NIGHT LIGHT` (£9,568.48), `ROUND SNACK BOXES SET OF4 WOODLAND` (£7,991.40), `SPACEBOY LUNCH BOX` (£7,485.60), `DOLLY GIRL LUNCH BOX` (£6,828.60), and `RED TOADSTOOL LED NIGHT LIGHT` (£3,479.40). This suggests a product-specific relationship or buying pattern that is worth investigating further.

> The pattern is not universal. Germany is the largest named market for `REGENCY CAKESTAND 3 TIER` (£9,061.95), while EIRE is also substantial for that product (£7,793.25). The combined smaller-market group leads some other products, but that does not identify one country as the leader because several countries are being added together.

> I therefore treat this chart as a prioritisation clue rather than proof of a broad national preference. The Netherlands result deserves closer account-level investigation, but my later customer check shows that 98.3% of Netherlands recorded revenue comes from customer `14646`. I cannot claim that Dutch customers generally buy these products. I would next compare units, order size, repeat behaviour, customer type, fulfilment cost, and margin before recommending a market action. Recorded revenue is not profit.

### Markdown before code cell 16 — Question 2: which markets are busy and which are valuable?

> I compare countries using recorded revenue and the number of distinct invoices. Transaction volume and revenue answer different questions, so I do not treat one as a substitute for the other. I pay particular attention to the visible Netherlands result because it was part of my original hypothesis.

### Code cell 16 — country revenue and activity table

```python
# I group the clean table by country and keep both revenue and activity visible.
country_summary = (
    df_clean.groupby("Country")
    .agg(
        Revenue=("Revenue", "sum"),
        Transactions=("InvoiceNo", "nunique"),
        Rows=("InvoiceNo", "size"),
    )
    .sort_values("Revenue", ascending=False)
)

display(country_summary.head(10))
display(country_summary.loc[country_summary.index.isin(["United Kingdom", "Netherlands"])])
```

### Markdown before code cell 17 — country chart

> I use Seaborn for the top-country revenue comparison. I keep the transaction-count comparison in the table because one chart should not carry every metric at once.

### Code cell 17 — Seaborn country chart

```python
# I keep the country chart focused on revenue; transaction counts remain in the table.
top_countries = country_summary.head(10).reset_index()

plt.figure(figsize=(10, 6))
sns.barplot(data=top_countries, x="Revenue", y="Country", color="seagreen")
plt.title("Top 10 countries by recorded revenue")
plt.xlabel("Recorded revenue")
plt.ylabel("Country")
plt.tight_layout()
plt.show()
```

### Markdown before code cell 18 — international country detail without the UK

> The full country chart confirms overall scale, but the United Kingdom makes the international bars difficult to compare. I create a second country view without the UK so the relative position of Netherlands, EIRE, Germany, France, and other international markets is visible.

### Code cell 18 — non-UK country revenue chart

```python
# I calculate the country comparison again without changing country_summary or df_clean.
non_uk_country_summary = (
    df_non_uk.groupby("Country")["Revenue"]
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .reset_index()
)

display(non_uk_country_summary)

# I use Seaborn to show international country scale without the UK dominating the axis.
plt.figure(figsize=(10, 6))
sns.barplot(
    data=non_uk_country_summary,
    x="Revenue",
    y="Country",
    color="seagreen",
)
plt.title("Top 10 countries by recorded revenue outside the UK")
plt.xlabel("Recorded revenue outside the UK")
plt.ylabel("Country")
plt.tight_layout()
plt.show()
```

### Markdown before code cell 19 — Question 2A: is Netherlands valuable per transaction?

> The country table shows that Netherlands is second in recorded revenue but has far fewer transactions than EIRE, Germany, France, or the United Kingdom. I now calculate revenue per distinct invoice so I can compare market value at the level of a sale rather than only at total-country scale.

> I use the label `EIRE` because that is how Ireland appears in this dataset. Revenue per transaction is a recorded-revenue measure, not profit or margin. It is a reason to investigate product mix, customer type, and costs more closely.

### Code cell 19 — selected-market value comparison

```python
# I select the markets needed to investigate the Netherlands result against useful comparators.
comparison_countries = [
    "Netherlands", "EIRE", "Germany", "France", "United Kingdom"
]

market_comparison = country_summary.loc[comparison_countries].copy()

# I calculate the average recorded revenue attached to one distinct invoice.
market_comparison["RevenuePerTransaction"] = (
    market_comparison["Revenue"] / market_comparison["Transactions"]
)

# I keep the exact measures visible before I interpret the comparison.
display(market_comparison[
    ["Revenue", "Transactions", "RevenuePerTransaction", "Rows"]
].sort_values("RevenuePerTransaction", ascending=False))
```

### Markdown before code cell 20 — revenue per transaction chart

> I visualise revenue per transaction for the selected markets. This chart directly tests whether the Netherlands result is driven by unusually high value per recorded sale.

### Code cell 20 — Seaborn revenue-per-transaction chart

```python
# I sort the bars so the highest recorded revenue per transaction is easiest to see.
market_plot = (
    market_comparison
    .reset_index()
    .sort_values("RevenuePerTransaction")
)

plt.figure(figsize=(10, 5))
sns.barplot(
    data=market_plot,
    x="RevenuePerTransaction",
    y="Country",
    color="darkorange",
)
plt.title("Recorded revenue per transaction in selected markets")
plt.xlabel("Recorded revenue per distinct invoice")
plt.ylabel("Country")
plt.tight_layout()
plt.show()
```

### Markdown before code cell 21 — transaction-value distribution

> An average can be pulled upward by a small number of large invoices. I therefore compare the median, upper quartile, and 90th percentile of transaction revenue, then inspect the distribution. This tells me whether the Netherlands pattern is broad or concentrated in a few high-value sales.

### Code cell 21 — transaction revenue distribution

```python
# I calculate one recorded-revenue value for each country and invoice.
transaction_values = (
    df_clean[df_clean["Country"].isin(comparison_countries)]
    .groupby(["Country", "InvoiceNo"], as_index=False)
    .agg(TransactionRevenue=("Revenue", "sum"))
)

# I calculate comparable summary statistics before plotting the distributions.
transaction_summary = (
    transaction_values.groupby("Country")["TransactionRevenue"]
    .agg(Median="median", Mean="mean")
    .reindex(comparison_countries)
)
transaction_summary["UpperQuartile"] = (
    transaction_values.groupby("Country")["TransactionRevenue"]
    .quantile(0.75)
    .reindex(comparison_countries)
)
transaction_summary["P90"] = (
    transaction_values.groupby("Country")["TransactionRevenue"]
    .quantile(0.90)
    .reindex(comparison_countries)
)

display(transaction_summary)

# I use a logarithmic y-axis because transaction values span a wide range.
plt.figure(figsize=(10, 6))
sns.boxplot(
    data=transaction_values,
    x="Country",
    y="TransactionRevenue",
    order=comparison_countries,
)
plt.yscale("log")
plt.title("Distribution of recorded revenue per transaction")
plt.xlabel("Country")
plt.ylabel("Recorded revenue per invoice (log scale)")
plt.xticks(rotation=25)
plt.tight_layout()
plt.show()
```

### Markdown before code cell 22 — Netherlands concentration check

> High-ticket transactions can be commercially interesting because product mix and margin may deserve closer investigation. They do not prove higher margin. Before I call the Netherlands a broad market opportunity, I check whether the recorded revenue is spread across customers or concentrated in one customer relationship.

### Code cell 22 — Netherlands customer concentration

```python
# I summarise Netherlands revenue and activity by customer.
netherlands_customer_summary = (
    df_clean[df_clean["Country"] == "Netherlands"]
    .groupby("CustomerID")
    .agg(
        Revenue=("Revenue", "sum"),
        Transactions=("InvoiceNo", "nunique"),
    )
    .sort_values("Revenue", ascending=False)
)

# I calculate each customer's share of Netherlands recorded revenue.
netherlands_customer_summary["RevenueShare"] = (
    netherlands_customer_summary["Revenue"]
    / netherlands_customer_summary["Revenue"].sum()
)

display(netherlands_customer_summary)

# I show the small customer population so concentration is visible rather than hidden in an average.
customer_plot = netherlands_customer_summary.reset_index()

# I treat customer IDs as category labels so Seaborn uses them on the y-axis.
customer_plot["CustomerID"] = customer_plot["CustomerID"].astype("string")

plt.figure(figsize=(10, 5))
sns.barplot(data=customer_plot, x="Revenue", y="CustomerID", color="mediumpurple")
plt.title("Netherlands recorded revenue by customer")
plt.xlabel("Recorded revenue")
plt.ylabel("Customer ID")
plt.tight_layout()
plt.show()
```

### Markdown-only Netherlands opportunity case

> The visible evidence makes Netherlands a credible relationship-level opportunity. It has the second-highest recorded revenue, but only 93 distinct transactions. Its recorded revenue per transaction is much higher than the selected comparison markets. That supports a closer look at product mix, order size, customer type, fulfilment cost, and margin.

> The concentration check identifies the main opportunity. Customer `14646` contributes 98.3% of Netherlands recorded revenue and most of its transactions. The order volume is consistent with a wholesale-style relationship, but this dataset does not contain a customer-type field. I therefore identify one dominant customer relationship rather than claim that Dutch customers generally buy high-ticket products.

> This remains an opportunity for growth through account development, retention, and understanding the relationship. It is not evidence of a general Netherlands market trend. My next evidence check would be product-level margin, customer type, repeat behaviour, and whether the other Netherlands customers show the same order-value pattern. Revenue per transaction is a useful opportunity signal, but it is not a margin measure.

### Markdown before code cell 23 — Question 3: how does recorded revenue change by month?

> I group the clean table by month to look for peaks, quieter periods, and the limits of this time comparison. The result describes recorded revenue in this dataset; it does not prove a cause for any rise or fall.

### Code cell 23 — monthly revenue table and Seaborn trend

```python
# I group recorded revenue by month to reveal the shape of the observed time period.
monthly_revenue = (
    df_clean.groupby("Month")["Revenue"]
    .sum()
    .reset_index()
)

display(monthly_revenue)

# I use Seaborn for the monthly trend after displaying the table of values.
plt.figure(figsize=(12, 5))
sns.lineplot(data=monthly_revenue, x="Month", y="Revenue", marker="o")
plt.title("Recorded revenue by month")
plt.xlabel("Month")
plt.ylabel("Recorded revenue")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Markdown-only analysis after code cell 23 — what does the monthly overview tell me?

> The monthly overview shows an April trough, a May rebound, a fairly steady June-to-August period, and a clear acceleration from September into November. November is the highest complete month in the comparable period. I do not interpret the fall in December 2011 as a New Year decline because the dataset ends on 9 December 2011. December is incomplete.

### Markdown before code cell 24 — Question 3A: is the movement driven by order volume or order value?

> I decompose monthly recorded revenue into invoices, quantity, active customers, and recorded revenue per invoice. This helps me distinguish a broad increase in demand from a small number of unusually valuable orders.

### Code cell 24 — monthly driver comparison

```python
# I calculate the main monthly drivers behind recorded revenue.
monthly_metrics = (
    df_clean.groupby("Month")
    .agg(
        Revenue=("Revenue", "sum"),
        Invoices=("InvoiceNo", "nunique"),
        Quantity=("Quantity", "sum"),
        ActiveCustomers=("CustomerID", "nunique"),
    )
    .reset_index()
    .sort_values("Month")
)

# I calculate the recorded revenue attached to one distinct invoice.
monthly_metrics["RevenuePerInvoice"] = (
    monthly_metrics["Revenue"] / monthly_metrics["Invoices"]
)

display(monthly_metrics)

# I use separate Seaborn panels because revenue and invoice counts have different units.
fig, axes = plt.subplots(2, 1, figsize=(12, 9), sharex=True)

sns.lineplot(
    data=monthly_metrics,
    x="Month",
    y="Revenue",
    marker="o",
    color="steelblue",
    ax=axes[0],
)
axes[0].set_title("Monthly recorded revenue")
axes[0].set_xlabel("")
axes[0].set_ylabel("Recorded revenue")

sns.lineplot(
    data=monthly_metrics,
    x="Month",
    y="Invoices",
    marker="o",
    color="darkorange",
    ax=axes[1],
)
axes[1].set_title("Monthly distinct invoices")
axes[1].set_xlabel("Month")
axes[1].set_ylabel("Distinct invoices")
axes[1].tick_params(axis="x", rotation=45)

plt.tight_layout()
plt.show()
```

### Markdown-only conclusion after code cell 24

> The November increase is mainly volume-led. Compared with October, November has more distinct invoices, more active customers, and more quantity, while recorded revenue per invoice is slightly lower. I therefore interpret the peak as a broad increase in order activity rather than a simple jump in average order value. This is useful for capacity, stock, and staffing decisions, but it does not explain the business cause.

### Markdown before code cell 25 — Question 3B: when does the Christmas ramp begin?

> I zoom in to weekly revenue and invoice counts from August through November. I exclude December because the available December data is incomplete. This gives the Commercial Director a more useful lead time for stock and operational planning.

### Code cell 25 — weekly ramp into the November peak

```python
# I create weekly measures so the timing of the autumn increase is visible.
weekly_metrics = (
    df_clean.assign(
        WeekStart=df_clean["InvoiceDate"].dt.to_period("W").dt.start_time
    )
    .groupby("WeekStart")
    .agg(
        Revenue=("Revenue", "sum"),
        Invoices=("InvoiceNo", "nunique"),
        ActiveCustomers=("CustomerID", "nunique"),
    )
    .reset_index()
)

# I use August to November as the comparable lead-in to the Christmas period.
weekly_ramp = weekly_metrics[
    (weekly_metrics["WeekStart"] >= "2011-08-01")
    & (weekly_metrics["WeekStart"] <= "2011-11-28")
].copy()

display(weekly_ramp)

fig, axes = plt.subplots(2, 1, figsize=(12, 9), sharex=True)

sns.lineplot(
    data=weekly_ramp,
    x="WeekStart",
    y="Revenue",
    marker="o",
    color="seagreen",
    ax=axes[0],
)
axes[0].set_title("Weekly recorded revenue before and during the November peak")
axes[0].set_xlabel("")
axes[0].set_ylabel("Recorded revenue")

sns.lineplot(
    data=weekly_ramp,
    x="WeekStart",
    y="Invoices",
    marker="o",
    color="mediumpurple",
    ax=axes[1],
)
axes[1].set_title("Weekly distinct invoices")
axes[1].set_xlabel("Week starting")
axes[1].set_ylabel("Distinct invoices")
axes[1].tick_params(axis="x", rotation=45)

plt.tight_layout()
plt.show()
```

### Markdown-only conclusion after code cell 25

> The weekly view shows the ramp beginning in September rather than suddenly appearing in November. Weekly revenue is mostly around £147,000–£173,000 in August, rises through September, becomes more variable but remains elevated in October, and reaches roughly £303,000–£373,000 through November. The practical implication is to prepare before September and not wait for the November peak to become visible.

### Markdown before code cell 26 — Question 3C: which products create the seasonal lift?

> I compare product revenue across Spring, Summer, Autumn, and the November peak. I use a heatmap because it lets me see persistent products and products whose revenue is concentrated in one period. I exclude December from this comparison because it is incomplete.

### Code cell 26 — seasonal product mix heatmap

```python
# I keep only comparable 2011 periods and assign each month to an analytical season.
seasonal_data = df_clean[
    df_clean["InvoiceDate"].dt.month.isin([3, 4, 5, 6, 7, 8, 9, 10, 11])
].copy()
seasonal_data["Season"] = seasonal_data["InvoiceDate"].dt.month.map(
    {
        3: "Spring",
        4: "Spring",
        5: "Spring",
        6: "Summer",
        7: "Summer",
        8: "Summer",
        9: "Autumn",
        10: "Autumn",
        11: "November peak",
    }
)

# I calculate product revenue for each seasonal period.
season_product_revenue = (
    seasonal_data.groupby(["StockCode", "Description", "Season"])["Revenue"]
    .sum()
    .reset_index()
)

# I keep stock code with description so similar descriptions remain distinguishable.
season_product_revenue["Product"] = (
    season_product_revenue["StockCode"]
    + " — "
    + season_product_revenue["Description"]
)

season_order = ["Spring", "Summer", "Autumn", "November peak"]
top_season_products = (
    season_product_revenue.groupby("Product")["Revenue"]
    .sum()
    .sort_values(ascending=False)
    .head(10)
    .index
)

season_product_matrix = (
    season_product_revenue[season_product_revenue["Product"].isin(top_season_products)]
    .pivot_table(
        index="Product",
        columns="Season",
        values="Revenue",
        aggfunc="sum",
        fill_value=0,
    )
    .reindex(index=top_season_products, columns=season_order, fill_value=0)
)

display(season_product_matrix)

# I use a Seaborn heatmap to show persistent and concentrated product demand.
plt.figure(figsize=(13, 8))
sns.heatmap(
    season_product_matrix,
    annot=True,
    fmt=".0f",
    cmap="YlGnBu",
    linewidths=0.5,
)
plt.title("Recorded revenue for leading products across comparable periods")
plt.xlabel("Period")
plt.ylabel("Product")
plt.tight_layout()
plt.show()
```

### Markdown-only conclusion after code cell 26

> The heatmap separates evergreen products from seasonal products. Regency Cake Stand 3 Tier and White Hanging Heart T-Light Holder perform across several periods, while Paper Chain Kit 50's Christmas strengthens sharply into November. Rabbit Night Light also rises strongly into the November peak. Picnic Basket Wicker 60 Pieces is concentrated in Summer. I would use these patterns to plan different stock and campaign calendars rather than treating every product as a year-round seller.

### Markdown before code cell 27 — Question 3D: is the peak domestic or international?

> I compare the United Kingdom with all other countries using monthly revenue share rather than raw revenue. The United Kingdom is much larger, so a raw-revenue chart would flatten the international pattern.

### Code cell 27 — UK and international revenue share

```python
# I compare market groups over the complete comparable period from March to November 2011.
comparable_market_data = df_clean[
    (df_clean["InvoiceDate"] >= "2011-03-01")
    & (df_clean["InvoiceDate"] < "2011-12-01")
].copy()

comparable_market_data["MarketGroup"] = comparable_market_data["Country"].where(
    comparable_market_data["Country"] == "United Kingdom",
    "Outside UK",
)

market_mix = (
    comparable_market_data.groupby(["Month", "MarketGroup"])["Revenue"]
    .sum()
    .reset_index()
    .sort_values("Month")
)

# I calculate each market group's share of its month's recorded revenue.
market_mix["RevenueShare"] = (
    market_mix["Revenue"]
    / market_mix.groupby("Month")["Revenue"].transform("sum")
)

display(market_mix)

# I use share rather than raw revenue so the international line remains visible.
plt.figure(figsize=(12, 5))
sns.lineplot(
    data=market_mix,
    x="Month",
    y="RevenueShare",
    hue="MarketGroup",
    marker="o",
    palette={"United Kingdom": "steelblue", "Outside UK": "darkorange"},
)
plt.title("Monthly recorded-revenue share: UK compared with outside the UK")
plt.xlabel("Month")
plt.ylabel("Share of monthly recorded revenue")
plt.ylim(0, 1)
plt.yticks([0, 0.25, 0.5, 0.75, 1], ["0%", "25%", "50%", "75%", "100%"])
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### Markdown-only conclusion after code cell 27

> The November peak is primarily domestic. The United Kingdom contributes about 88% of November recorded revenue, while outside-UK markets contribute about 12%. International share is more visible in some earlier months, reaching about 20% in August. The Commercial Director should therefore plan the November capacity response mainly around UK demand, while treating international demand as a separate opportunity rather than assuming that every market follows the UK Christmas pattern.

## Part 5 — findings, AI record, and review

### Findings from the completed analysis

> I found that the cleaning decisions produce a usable paid-product analysis table while preserving the raw and excluded audit views. The final table contains 522,540 rows and has complete `CustomerID` coverage within this cleaned scope. That makes customer-level analysis possible, but it does not tell me whether a customer is a consumer, retailer, wholesaler, or another type of buyer.

> My first business finding is that the United Kingdom dominates recorded revenue, so I use UK-excluded views when I need to compare international markets. The Netherlands is a relationship-level opportunity signal, but 98.3% of its recorded revenue comes from customer `14646`. I therefore do not claim that Dutch customers generally buy high-ticket products.

> My second finding is that the November peak is mainly volume-led and domestic. The weekly ramp begins in September, and November has more invoices and active customers rather than simply a higher average invoice value. The United Kingdom contributes about 88% of November recorded revenue. December 2011 is incomplete, so I do not describe it as a confirmed New Year decline.

> My third finding is that product demand has different seasonal shapes. Some products perform across several periods, while Christmas paper chains and other products strengthen into November, and picnic-basket products are concentrated in Summer. This gives the Commercial Director a basis for different stock and campaign calendars.

> I keep these findings descriptive. Recorded revenue is not profit, timing does not prove cause, and customer identifiers do not reveal customer type, margin, shipping cost, or business health.

### Limitations and scope

> I excluded duplicate rows, cancellation-style records, negative-quantity adjustment rows, operational non-product lines, and non-positive-price rows from the primary paid-sales table according to the decision checkpoint. The excluded rows remain available for audit, but the findings describe the cleaned paid-product scope rather than every raw transaction-like row.

> The dataset covers 1 December 2010 to 9 December 2011, with an incomplete final December. It contains country and customer identifiers but no product cost, shipping cost, profit, customer-type, shipping-address, or marketing-campaign fields. I cannot use this dataset alone to claim margin, causation, drop-shipping, or the health of a customer's business.

### Code review

> I simplified the earlier dense workflow into named stages: raw extraction, pre-cleaning investigation, decision checkpoint, working copy, cleaning, validation, business measures, and focused questions. I kept the raw table unchanged and retained excluded rows so the cleaning process remains auditable.

> I used named Boolean masks because each cleaning rule needs to be counted, inspected, and explained separately. I used Pandas grouping and aggregation to calculate revenue, activity, customer concentration, and time measures. I used Seaborn for the analytical charts and Matplotlib where subplot layout or the raw-versus-clean comparison needed more control.

> I deliberately did not use Plotly. The assessment and this dataset are well served by reproducible static charts, and dynamic interaction would not add analytical value here. I ran the notebook from top to bottom and checked the visible tables, charts, and validation outputs rather than treating successful execution alone as proof.

### AI use

> Generative AI helped me evaluate the complexity and nuance of the datasets offered as choices.  This evaluation and overview was not however done by loading the actual datasets into context.  It was done by referencing Kaggle and the datasets by name only.  I chose #4 Online Retail because it was historically the most messy and troublesome of the datasets. Then during my initial exploration of the actual dataset I spotted what I thought was an anomaly worth investigating.  A mystery Whale customer who was the single largest source of income for the businees.  I intended to perform a forensic review of this mystery customer to identify spending patterns, try to understand the nature of their business, and develop strategies to maximise the offering to this customer.  I grew suspicious when examiming my data overview, decided to check this customer ID, and discovered my target was an artefact created by a change in the dataset, that cleaned up entries with no Customer ID, into a single identity as a placeholder.  My target was a ghost.

>I had to pivot to a more standard investigation, of spending patterns across products, regions, and timeframes.  My new plan laid out I then asked Codex to review my proposed cell sequence, to simplify and explain any unfamiliar code it suggested, discuss chart forms, and draft possible interpretations. I remained responsible for the analysis angle decisions, the actual code (I rejected anything which I could not read and understand), and all conclusions derived from the analysis.

> I checked suggestions against the visible notebook output and changed or rejected them when the evidence did not support them. For example, I rejected the Product Graveyard, panic-buying, drop-shipping, and phantom-catalog stories as conclusions because the dataset did not provide enough signal or the required fields. I retained only the narrower questions that the data could support, such as seasonal demand, customer concentration, market comparison, and bulk-purchasing behaviour as a possible future investigation.

> I used AI - Claude - Codex - and my Gemini Gem AI Kevin as sounding boards against which I could bounce ideas, and ask questions.  When they grew overeaged (often) I would reject their more elablorate suggestions. I wanted this project to be mine, and to be evidence led, not narrative led.  I kept first-person explanations and recorded where the evidence limited the story.

### Next questions

> I would next investigate customer retention and repeat behaviour where identifiers permit it, using cohorts or time-based customer summaries rather than assuming that every customer is a consumer.

> If cost and fulfilment data became available, I would compare revenue with margin and shipping cost before making product or market recommendations. I would also consider a simple stock-planning or anomaly-prioritisation model only after the descriptive measures had been validated over a longer and more complete time window.

> I would not build a predictive model from this dataset alone. The next useful improvement is better business context and additional fields, not more complex code.

## Assessment coverage retained by this simpler route

- **LO1:** business intent, retail analytics applications, three focused questions, and a clearly labelled future AI opportunity.
- **LO2:** Python, Pandas, NumPy import, Matplotlib, Seaborn, cleaning, transformation, tables, charts, code review, and readable technique choices.
- **LO3:** explicit generative-AI record, visible-output checking, AI limitation, and alternative future analysis.
- **LO4:** raw extraction, inspection, documented cleaning decisions, processing, validation, excluded-row audit view, and saved clean-table hand-off.

## Markdown preservation register

I keep the exact Markdown backup beside this proposal: `Online_Retail_Analysis_Markdown_Backup_20260825.md`. When I build the replacement notebook, I copy or lightly adapt the existing first-person passages rather than replacing them with generic third-person wording.

The passages I protect especially are:

- the opening business context and hypothesis;
- the investigation questions before the decision checkpoint;
- the explanation that cleaning is reversible and does not replace my decisions;
- my evidence-led analysis notes and overlap caveat;
- my code-review and AI-disclosure wording;
- my limitations and future questions;
- my personal “Happy Coder” sign-off and other genuine reactions where they support the story.

## Approval gate

I built the simpler notebook only after I have approved the cell sequence and the preserved Markdown wording. Only visualisations that told an interesting or useful story were to be selected.  My mantra, and the one key project instruction I created for all AI helpers was:

> *“For every analysis cell, write the human question and metric in the Markdown above it. A chart is not a question by itself.”*


