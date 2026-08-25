# Online Retail Analysis — notebook build handover

## Continue from here

This handover is for the next Codex instance working on the actual notebook build. Keep the work focused on the single linear Jupyter Notebook. Do not restart the project or redesign the structure.

## Canonical files

- **Cell plan:** `Notebook_Cell_Plan.md`
- **Live notebook:** `jupyter_notebooks/Online_Retail_Analysis.ipynb`
- **Raw dataset:** `Data_Sets/Online_Retail.csv`

The cell plan is the source of truth for the intended notebook sequence. The live notebook is Phil's learner-controlled working file and must be changed carefully, with a timestamped backup before any edit.

## Current analytical direction

The notebook is one continuous workflow:

1. setup and extraction;
2. ETL and evidence-led data-quality decisions;
3. exploratory analysis and visualisation; and
4. findings, limitations, and the next question.

The imagined audience is a Commercial Director who wants evidence about:

- potential markets for growth;
- possible underperformance signals; and
- product, country, time, and customer patterns.

The Netherlands is an investigation question, not a conclusion. Any claim must be supported by output displayed in the notebook.

## Exact current state

- Phase 1 is present and includes the business intent.
- Cells 1–4 are present and have been worked through.
- Cell 5 established the initial quality questions.
- Cell 5A investigated the 1,336 negative-quantity rows whose invoice numbers do not start with `C`. The visible rows and descriptions suggest a returns/damage-related group, but conclusions must remain limited to what the displayed output supports.
- Cell 5B examined non-numeric-leading stock codes.
- Cell 5B1 expanded that check to the complete population, including descriptions, counts, prices, quantity flags, and missing descriptions.
- The working interpretation recorded so far is that the population is mixed. Postage-related groups appear prominent by count, but a blanket removal decision is not justified from counts alone.

## Important sequencing fact

The **plan already contains Cell 5B2**, including its Markdown and code. The live notebook was checked on 25 August 2026 and does **not** yet contain Cell 5B2: it currently moves from the Cell 5B1 analysis to Cell 6.

Therefore the next notebook-build task is:

1. insert the Cell 5B2 Markdown cell from the plan;
2. insert the Cell 5B2 code cell immediately below it;
3. run it and inspect the complete visible output; and
4. write an evidence-led conclusion before continuing to Cell 5C.

Do not skip directly to standardisation or cleaning. Cell 5B2 exists to measure the analytical impact of the identified non-numeric stock-code groups using:

- row counts and share of all rows;
- total quantity;
- provisional line revenue (`Quantity × UnitPrice`); and
- share of total provisional line revenue.

The exact copy-ready material is in `Notebook_Cell_Plan.md` under **Cell 5B2 — measure the analytical impact of the identified groups**.

## Working rules

- **Golden evidence rule:** never extrapolate beyond displayed evidence. Show the rows, counts, categories, or calculations first; only then describe what they demonstrate. Anything not shown remains a question or hypothesis.
- Keep Markdown concise and directly above the code it explains.
- Use occasional `#` comments inside code cells to explain important operations.
- Preserve Phil's wording and analysis where it is already present; do not silently replace learner-authored conclusions.
- Do not introduce a second notebook or duplicate the setup cells.
- Matplotlib and Seaborn remain the core visualisation tools. Plotly is optional, not a requirement for the route currently planned.

## Immediate handoff instruction

Open `Notebook_Cell_Plan.md`, copy the planned Cell 5B2 Markdown and code into the live notebook **before the current Cell 6**, run the new code, and wait for Phil to inspect the output before drafting the conclusion or moving to Cell 5C.
