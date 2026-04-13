# Android App Store Analysis

Analysis of Google Play Store app data to uncover category competition, pricing strategies, and install patterns using Plotly visualisations.

This project performs a comprehensive analysis of the Android app market by comparing thousands of apps across the Google Play Store. It answers concrete business questions: which categories are most competitive, which offer the strongest download opportunities, how much revenue a paid app can realistically expect, and how many installations a developer gives up by choosing to charge for their app. The analysis mirrors the kind of market intelligence produced by firms such as App Annie and Sensor Tower.

The dataset was scraped from the Google Play Store by Lavanya Gupta in 2018 and contains 10,841 rows and 12 columns covering app name, category, rating, review count, size, install count, type (free/paid), price, content rating, and genres. The raw data is cleaned by removing NaN rows and duplicate entries, and several columns require type conversion (commas stripped from Installs, dollar signs stripped from Price) before analysis can proceed.

No external APIs or credentials are required. All data is bundled as a local CSV file.

---

## Table of Contents

1. [Quick start](#1-quick-start)
2. [Analysis flow](#2-analysis-flow)
3. [Features](#3-features)
4. [Dataset schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Notebook reference](#6-notebook-reference)
7. [Configuration reference](#7-configuration-reference)
8. [Course context](#8-course-context)
9. [Dependencies](#9-dependencies)

---

## 1. Quick start

```bash
git clone https://github.com/xavier-oc-programming/android-app-store-analysis.git
cd android-app-store-analysis
pip install -r requirements.txt
jupyter notebook
```

Open `practice/A_Main_Analysis.ipynb` to run the full analysis end-to-end.

---

## 2. Analysis flow

```
data/apps.csv
    │
    ▼
pd.read_csv('../data/apps.csv')  →  df_apps  (10,841 × 12)
    │
    │  ── Cleaning ───────────────────────────────────────────────
    ├── .drop(['Last_Updated', 'Android_Ver'])          →  10 columns remain
    ├── .dropna()                                       →  9,367 rows
    ├── .drop_duplicates(subset=['App','Type','Price'])  →  8,199 rows  →  df_apps_clean
    │
    │  ── Type Conversion ────────────────────────────────────────
    ├── str.replace(',','') + pd.to_numeric()           →  Installs as int
    ├── str.replace('$','') + pd.to_numeric()           →  Price as float
    │
    │  ── Ranking ────────────────────────────────────────────────
    ├── .sort_values('Rating')                          →  highest-rated apps
    ├── .sort_values('Reviews')                         →  most-reviewed apps
    │
    │  ── Aggregation ────────────────────────────────────────────
    ├── .value_counts() on Content_Rating               →  content rating counts
    ├── .groupby('Category').agg(count, sum) + merge()  →  apps vs installs per category
    ├── .str.split(';').stack().value_counts()           →  genre frequency counts
    ├── .groupby(['Category','Type']).agg(count)        →  free vs paid app counts
    ├── filter Type=='Paid', Price × Installs           →  revenue estimates
    │
    │  ── Visualisation ──────────────────────────────────────────
    ├── px.pie()                                        →  content rating distribution
    ├── px.scatter()                                    →  category concentration
    ├── px.bar(color_continuous_scale='Agsunset')       →  top 15 genres
    ├── px.bar(barmode='group')                         →  free vs paid per category
    ├── px.box(y='Installs', x='Type')                  →  download loss for paid apps
    ├── px.box(x='Category', y='Revenue Estimate')      →  revenue by category
    └── px.box(x='Category', y='Price')                 →  pricing strategy by category
```

---

## 3. Features

- **Category competition** — scatter plot of number of apps vs total installs per category reveals crowded vs opportunity-rich segments
- **Content rating breakdown** — pie / donut chart of audience age distribution across all apps
- **Genre popularity** — horizontal bar chart of the top 15 genres with colour scale
- **Free vs paid split** — grouped bar chart showing free and paid app counts per category
- **Download loss for paid apps** — box plot comparing installs distribution for free vs paid apps
- **Revenue estimates by category** — box plot of `Price × Installs` per category for paid apps
- **Pricing strategy** — box plot of paid app prices by category, ordered by maximum price

---

## 4. Dataset schema

### `data/apps.csv`

| Column | Type | Description |
|---|---|---|
| App | string | App display name |
| Category | string | Primary store category (e.g. GAME, TOOLS) |
| Rating | float | User rating 1–5 (NaN if no ratings yet) |
| Reviews | int | Total number of user reviews |
| Size_MBs | float | App size in megabytes |
| Installs | string → int | Install count range (e.g. `10,000+`); converted to int after cleaning |
| Type | string | `Free` or `Paid` |
| Price | string → float | Price in USD (e.g. `$0.99`); converted to float after stripping `$` |
| Content_Rating | string | Audience age group (e.g. `Everyone`, `Teen`) |
| Genres | string | Semi-colon-separated genre list (e.g. `Action;Action & Adventure`) |
| Last_Updated | string | Date string — dropped during cleaning |
| Android_Ver | string | Minimum Android version — dropped during cleaning |

**Computed columns added at runtime:**

| Column | Description |
|---|---|
| Revenue Estimate | `Price × Installs` — ballpark gross revenue for paid apps |
| Genre_1 / Genre_2 | Primary and secondary genre split from the Genres column |

---

## 5. Architecture

```
android-app-store-analysis/
│
├── theory/                         # Lesson notes — concepts and annotated methods
│   ├── 00__Overview.ipynb          # Day 76 goals and project outline
│   ├── 01__Data_Cleaning.ipynb     # dropna, drop_duplicates
│   ├── 02__Preliminary_Exploration.ipynb  # sort_values for rating, size, reviews
│   ├── 03__Pie_and_Donut_Charts.ipynb     # px.pie(), content rating distribution
│   ├── 04__Numeric_Type_Conversions.ipynb # str.replace + pd.to_numeric
│   ├── 05__Bar_Charts_and_Scatter_Plots.ipynb  # px.bar, px.scatter, category analysis
│   ├── 06__Extracting_Nested_Data.ipynb   # str.split + stack vs explode
│   ├── 07__Grouped_Bar_Charts_and_Box_Plots.ipynb  # px.box, grouped bar, revenue
│   ├── 08__Summary.ipynb           # Learning points recap
│   └── ZZ__Agg_Tutorial.ipynb      # Deep dive into .agg() patterns
│
├── practice/                       # Student work — exercises and full analysis
│   └── A_Main_Analysis.ipynb       # Complete end-to-end Google Play Store analysis
│
├── data/                           # Seed dataset
│   └── apps.csv                    # Google Play Store data (8,199 rows after cleaning)
│
├── docs/
│   └── COURSE_NOTES.md             # Original exercise brief and key concept notes
│
├── requirements.txt                # Python dependencies with minimum versions
├── .gitignore
└── README.md
```

---

## 6. Notebook reference

### theory/

| Notebook | Key methods covered | Question answered |
|---|---|---|
| 00__Overview.ipynb | — | What will be built by end of day? |
| 01__Data_Cleaning.ipynb | `.drop()`, `.dropna()`, `.duplicated()`, `.drop_duplicates(subset=[...])` | How to clean 10k rows down to 8k usable entries |
| 02__Preliminary_Exploration.ipynb | `.sort_values()`, `.shape`, `.sample()` | Which apps have the highest ratings, most reviews, largest size? |
| 03__Pie_and_Donut_Charts.ipynb | `px.pie()`, `.value_counts()`, `update_traces()` | How are apps distributed by content rating? |
| 04__Numeric_Type_Conversions.ipynb | `str.replace()`, `pd.to_numeric()`, `.groupby().count()` | How many apps exceed 1B installs? What do apps cost? |
| 05__Bar_Charts_and_Scatter_Plots.ipynb | `px.bar()`, `px.scatter()`, `pd.merge()`, `.groupby().agg()` | Which categories are most competitive vs most popular? |
| 06__Extracting_Nested_Data.ipynb | `.str.split().stack()`, `.explode()`, `.value_counts()` | How many unique genres exist? What are the top 15? |
| 07__Grouped_Bar_Charts_and_Box_Plots.ipynb | `px.box()`, `px.bar(barmode='group')`, `.groupby(['Cat','Type'])` | Free vs paid split; revenue estimates; pricing by category |
| 08__Summary.ipynb | — | What were the key learning points from Day 76? |
| ZZ__Agg_Tutorial.ipynb | `.agg()` with dict, list, named aggregations | How does `.agg()` work across different use cases? |

### practice/

| Notebook | Key methods covered | Question answered |
|---|---|---|
| A_Main_Analysis.ipynb | Full pipeline from raw CSV to Plotly charts | All six business questions end-to-end |

---

## 7. Configuration reference

| Value | Location | Description |
|---|---|---|
| `'../data/apps.csv'` | `practice/A_Main_Analysis.ipynb` | Relative path from practice/ to the dataset |
| `pd.options.display.float_format = '{:,.2f}'.format` | `practice/A_Main_Analysis.ipynb` | Display floats with 2 decimal places and comma separators |
| `df_apps_clean = df_apps_clean[df_apps_clean["Price"] < 250]` | `practice/A_Main_Analysis.ipynb` | Filters out junk high-price apps above $250 |
| `color_continuous_scale='Agsunset'` | `practice/A_Main_Analysis.ipynb` | Colour scale used in the genre bar chart |
| `yaxis=dict(type='log')` | `practice/A_Main_Analysis.ipynb` | Log scale on the installs box plot y-axis |

---

## 8. Course context

100 Days of Code — Data Science Bootcamp, Day 76: Beautiful Plotly Charts and Analysing the Android App Store.

See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the full exercise brief and concept notes.

---

## 9. Dependencies

| Module | Used in | Purpose |
|---|---|---|
| pandas | practice/, theory/ | Data loading, cleaning, groupby, merge, type conversion |
| numpy | practice/ | Numeric operations (implicit via pandas) |
| plotly | practice/, theory/ (03–07) | Interactive pie, bar, scatter, and box charts |
| notebook | — | Jupyter notebook runtime |
