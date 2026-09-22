# Sales & Market Share Dashboard

A Power BI case study analyzing sales performance and competitive market share across five countries, built end-to-end from raw CSV/Excel sources through Power Query transformation, a star-schema data model, DAX measures, and a 3-page interactive report.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-5F7DF5?style=flat)
![Power Query](https://img.shields.io/badge/Power%20Query-683CEC?style=flat)

---

## Overview

This report simulates a sales and market share analysis for a manufacturing business. The report's owner (**Atlas**) is treated as one of 14 competing manufacturers, and the analysis covers both Atlas's internal performance and its position relative to competitors — across the USA, Canada, Germany, Japan, Mexico, and Nigeria.

**Assumption:** Atlas is the manufacturer whose performance is being tracked against the competitive field.

## Report Pages

| Page | Focus | Key visuals |
|---|---|---|
| **Market Share Analysis** | Who's winning, and by how much | KPI cards, ranked manufacturer table, drillable bar chart (Manufacturer → Region → Category), dynamic Top-5 highlighting |
| **Top N Filtering** | Leading products/regions, adjustable on the fly | What-If Parameter slider (1–20), two rank-driven bar charts |
| **Performance Comparison** | Trends over time, Atlas vs. the field | Revenue trend area chart, YoY growth indicator, 100% stacked Company-vs-Competitor bar, top-5 manufacturer trend lines, Decomposition Tree |

## Data Sources

| Source | Description |
|---|---|
| `Sales.csv` | Domestic (USA) sales transactions |
| `Canada.csv`, `Germany.csv`, `Japan.csv`, `Mexico.csv`, `Nigeria.csv` | International sales transactions, one file per country |
| `bi_dimensions.xlsx` → Product Details | Product, Segment, Category, Manufacturer link, Price |
| `bi_dimensions.xlsx` → Manufacturer | Manufacturer names (stored transposed in the source file) |
| `bi_dimensions.xlsx` → Geography | Zip/City/State/Region/District/Country reference, ~177K rows |

## Data Model

Star schema with one snowflake branch (Manufacturer → Product):

```
                Dim_Date
                   |
                   | (1) → (*)
                   |
Dim_Product ──(1)→(*)── Fact_Sales ──(*)→(1)── Dim_Geography
     |
     | (*)→(1)
     |
Dim_Manufacturer
```

- **Fact_Sales** — combined domestic + international transactions (~1.6M+ rows)
- **Dim_Geography** joins via a composite `Zip + Country` key, since zip codes are not globally unique
- **Dim_Date** is a generated calendar table, marked as the model's official Date Table

## Key DAX Measures

```dax
Market Share % = DIVIDE([Total Revenue], [Total Market Revenue])

Top 5 Manufacturer Flag (All-Time) =
VAR RankAllTime =
    RANKX(ALL(Dim_Manufacturer), CALCULATE([Total Revenue], ALL(Dim_Date)), , DESC, DENSE)
RETURN IF(RankAllTime <= 5, "Top 5", "Other")

YoY Growth % = DIVIDE([Total Revenue] - [Prior Year Revenue], [Prior Year Revenue])

Company Market Share % = DIVIDE([Company Revenue], [Total Revenue])
```

Full measure list and reasoning: see [`docs/Project_Documentation.pdf`](docs/Project_Documentation.pdf).

## Data Quality Notes

A few real issues were found and handled during the build rather than hidden:

- Domestic sales had no `Country` field — resolved by hardcoding `"USA"`.
- The Manufacturer sheet was stored transposed and required a Transpose step.
- ~10% of Geography rows (mostly Mexico) were missing `City` — backfilled from `District`.
- `Zip` needed to stay text-typed throughout to preserve leading US zeros and alphanumeric Canada/Japan codes.
- Zip codes aren't globally unique, so the Fact–Geography relationship uses a composite `Zip + Country` key.

## Tech Stack

- **Power BI Desktop** — data model, DAX, report
- **Power Query (M)** — ETL / data cleansing
- **Custom Power BI theme** — matched to the provided design reference

## Repository Structure

```
├── Infilect_Assignment.pbix      # Main Power BI report file
├── docs/
│   └── Project_Documentation.pdf # Full build write-up: sources, transformations, model, DAX, chart rationale
├── datasets/                     # Raw source files (Sales, international CSVs, bi_dimensions.xlsx)
└── README.md
```

## Opening the Report

1. Requires **Power BI Desktop** (Windows).
2. Open `Infilect_Assignment.pbix`.
3. If prompted, update the data source file paths under **Transform Data → Data source settings** to match where you've placed the `datasets/` folder locally.
4. Refresh to reload the model.

## Evaluation Criteria Mapping

| Criterion | Where it's addressed |
|---|---|
| Data modelling approach | Star schema, one-to-many relationships, hidden join keys, marked Date table |
| DAX correctness & performance | Dedicated measures table, context-safe ranking measures |
| Visual clarity & usability | Consistent custom theme, card-based layout, dynamic filtering |
| Analytical insights | Market share/rank framing, documented data-quality findings |
| Overall solution design | Three pages mapped directly to the brief's three required analyses |

---

*Built as a Power BI case study — see `docs/Project_Documentation.pdf` for the full step-by-step build log.*
