# Android App Store Analysis

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-2.0%2B-150458?logo=pandas&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-5.18%2B-3F4F75?logo=plotly&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3%2B-F7931E?logo=scikit-learn&logoColor=white)
![CI](https://img.shields.io/github/actions/workflow/status/xavier-oc-programming/android-app-store-analysis/publish_notebook.yml?branch=main&label=notebook%20publish&logo=github-actions&logoColor=white)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-live-22863a?logo=github&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue)

If you were launching an app today, where would you put it? This analysis of 8,199 Google Play Store apps maps category-level competition, install opportunity, and revenue potential to identify where the market is undersupplied — and where it is saturated.

The analytical approach runs in three parts: an Opportunity Score that measures demand relative to supply per category, a paid viability analysis that isolates categories where charging for an app actually produces meaningful revenue, and a K-Means segmentation that clusters all 8,199 individual apps into four distinct market positions using normalised features across installs, ratings, reviews, price, and app size. PCA reduces the five-dimensional feature space to two interpretable components for visualisation.

The most striking result from the segmentation is that Mass Market Free dominates the Play Store by volume — most apps compete in a high-install, zero-price environment where organic discovery drives growth. Premium Paid apps exist in a structurally separate cluster: low install count, high price, high rating. A new developer choosing between these two positions is making a fundamentally different bet on audience acquisition versus willingness to pay.

---

## Table of Contents

1. [Quick Start](#1-quick-start)
2. [Analysis Flow](#2-analysis-flow)
3. [Key Findings](#3-key-findings)
4. [Dataset Schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Visualisations](#6-visualisations)
7. [Operations Reference](#7-operations-reference)
8. [Background](#8-background)
9. [Dependencies](#9-dependencies)

---

## 1. Quick Start

```bash
git clone https://github.com/xavier-oc-programming/android-app-store-analysis.git
cd android-app-store-analysis
pip install -r requirements.txt
jupyter notebook
```

Open `notebooks/analysis/android_analysis.ipynb` to run the full analysis end-to-end.

The rendered notebook (no code, outputs only) is available at:
https://xavier-oc-programming.github.io/android-app-store-analysis/notebook_web_render/

---

## 2. Analysis Flow

```
data/apps.csv
    │
    ▼
pd.read_csv('../../data/apps.csv')  →  df_apps  (10,841 × 12)
    │
    │  ── Cleaning ───────────────────────────────────────────────
    ├── .drop(['Last_Updated', 'Android_Ver'])           →  10 columns remain
    ├── .dropna()                                        →  9,367 rows
    ├── .drop_duplicates(subset=['App','Type','Price'])   →  8,199 rows  →  df_apps_clean
    │
    │  ── Type Conversion ────────────────────────────────────────
    ├── str.replace(',','') + pd.to_numeric()            →  Installs as int
    ├── str.replace('$','') + pd.to_numeric()            →  Price as float
    │
    │  ── Ranking ────────────────────────────────────────────────
    ├── .sort_values('Rating')                           →  highest-rated apps
    ├── .sort_values('Reviews')                          →  most-reviewed apps
    │
    │  ── Core Aggregation ───────────────────────────────────────
    ├── .value_counts() on Content_Rating                →  content rating counts
    ├── .groupby('Category').agg(count, sum) + merge()   →  apps vs installs per category
    ├── .str.split(';').stack().value_counts()            →  genre frequency counts
    ├── .groupby(['Category','Type']).agg(count)         →  free vs paid app counts
    ├── filter Type=='Paid', Price × Installs            →  revenue estimates
    │
    │  ── Improvement 1: Opportunity Score ──────────────────────
    ├── total_installs / num_apps per category           →  opportunity_score
    ├── percentile thresholds (p25, p75)                 →  market_status labels
    │
    │  ── Improvement 2: Paid Viability ─────────────────────────
    ├── filter Price ≤ $29.99, Type=='Paid'              →  df_paid_viable
    ├── .groupby('Category').agg(median_revenue,         →  paid_cat summary
    │         median_price, paid_app_count)
    ├── filter paid_app_count ≥ 10                       →  credible categories only
    │
    │  ── Improvement 3: K-Means + PCA ──────────────────────────
    ├── log1p(Installs), log1p(Reviews) transforms       →  reduce skew
    ├── StandardScaler on 5 features                     →  X_scaled
    ├── KMeans(n_clusters=4, random_state=42)            →  Segment labels
    ├── PCA(n_components=2)                              →  X_pca for visualisation
    │
    │  ── Visualisation ──────────────────────────────────────────
    ├── px.pie()                                         →  content rating distribution
    ├── px.bar(orientation='h')                          →  category installs
    ├── px.scatter()                                     →  category concentration
    ├── px.bar(color_continuous_scale='Agsunset')        →  top 15 genres
    ├── px.bar(barmode='group')                          →  free vs paid per category
    ├── px.box(y='Installs', x='Type')                   →  download loss for paid apps
    ├── px.box(x='Category', y='Revenue Estimate')       →  revenue by category
    ├── px.box(x='Category', y='Price')                  →  pricing by category
    ├── px.bar(x='opportunity_score', color=score)       →  opportunity score ranking
    ├── px.scatter(x=median_price, y=median_revenue)     →  paid viability scatter
    └── px.scatter(PC1, PC2, color=Segment_Label)        →  K-Means PCA projection
```

All charts are saved to `plots/` as PNG at 150 dpi using `fig.write_image(scale=2)` (requires kaleido).

---

## 3. Key Findings

**Opportunity Score:** Categories are ranked by total installs divided by number of competing apps. The highest-scoring categories combine large aggregate download volumes with a relatively small number of entrants — they represent undersupplied demand. The lowest-scoring categories have the inverse profile: many apps competing for proportionally modest install volume. The `market_status` column in the summary DataFrame classifies each category as Opportunity (top quartile), Saturated (bottom quartile), or Competitive.

**Paid App Viability:** After filtering to apps priced at $29.99 or below (removes data-entry junk with no installs) and requiring at least 10 paid apps per category, only a subset of categories produce meaningful median revenue estimates. Professional tools and niche productivity categories outperform casual and entertainment categories by a wide margin. Most categories show near-zero median revenue for paid apps, confirming that free-with-ads is the commercially rational default for the majority of the market.

**K-Means App Segmentation:** Four clusters emerge from the normalised feature space. Mass Market Free captures the highest install volume and dominates by app count. Premium Paid is a structurally isolated cluster: low installs, above-average prices, high ratings — typically a niche professional or enthusiast audience. Hidden Gems sits at moderate installs with above-average ratings and low review counts, suggesting under-discovered apps with strong product quality. Struggling is characterised by low installs and below-average ratings. Free versus paid does not map cleanly onto segment boundaries — Hidden Gems contains predominantly free apps, which means product quality and niche focus drive segment membership more than pricing model alone.

---

## 4. Dataset Schema

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
| opportunity_score | `total_installs / num_apps` per category |
| market_status | `Opportunity` / `Competitive` / `Saturated` based on percentile thresholds |
| Segment | Integer cluster label (0–3) from KMeans |
| Segment_Label | Business label assigned from centroid inspection |

---

## 5. Architecture

```
android-app-store-analysis/
│
├── notebooks/
│   ├── analysis/
│   │   └── android_analysis.ipynb     # Full end-to-end analysis
│   └── concepts/                      # Concept notebooks: methods and annotated examples
│       ├── 00__Overview.ipynb
│       ├── 01__Data_Cleaning.ipynb
│       ├── 02__Preliminary_Exploration.ipynb
│       ├── 03__Pie_and_Donut_Charts.ipynb
│       ├── 04__Numeric_Type_Conversions.ipynb
│       ├── 05__Bar_Charts_and_Scatter_Plots.ipynb
│       ├── 06__Extracting_Nested_Data.ipynb
│       ├── 07__Grouped_Bar_Charts_and_Box_Plots.ipynb
│       ├── 08__Summary.ipynb
│       └── ZZ__Agg_Tutorial.ipynb
│
├── data/
│   └── apps.csv                       # Google Play Store dataset (10,841 rows raw)
│
├── plots/                             # All charts — saved at 150 dpi on notebook run
│
├── notebook_web_render/
│   └── index.html                     # Rendered notebook (no code) for GitHub Pages
│
├── docs/
│   └── COURSE_NOTES.md               # Concept notes (untouched)
│
├── .github/workflows/
│   └── publish_notebook.yml           # CI: auto-renders notebook to HTML on push
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 6. Visualisations

Charts are saved to `plots/` as PNG at 150 dpi via `fig.write_image(scale=2)` when the notebook runs.

| File | Description |
|---|---|
| `content_rating_donut.png` | Donut chart of content rating distribution |
| `content_rating_pie.png` | Pie chart of content rating distribution (labelled) |
| `top_10_categories_bar.png` | Top 10 categories by number of apps |
| `category_installs_bar.png` | All categories ranked by total installs (horizontal) |
| `category_popularity_bar.png` | Category popularity with labelled axes |
| `category_concentration_scatter.png` | Apps vs installs per category — scatter with size |
| `top_genres_bar.png` | Top 15 genres by frequency, coloured by count |
| `free_vs_paid_bar.png` | Free vs paid app count per category (grouped bar) |
| `installs_free_vs_paid_box.png` | Install distribution: free vs paid (log scale box) |
| `revenue_by_category_box.png` | Revenue estimate distribution by category (paid apps) |
| `price_by_category_box.png` | Price distribution by category (paid apps) |
| `opportunity_score_by_category.png` | All categories ranked by Opportunity Score |
| `paid_app_viability_scatter.png` | Median price vs median revenue, sized by paid app count |
| `app_segments_pca.png` | PCA projection of K-Means segments with centroid markers |

---

## 7. Operations Reference

| Value | Location | Description |
|---|---|---|
| `'../../data/apps.csv'` | `notebooks/analysis/android_analysis.ipynb` | Relative path from notebooks/analysis/ to dataset |
| `pd.options.display.float_format = '{:,.2f}'.format` | analysis notebook | Display floats with 2 decimal places |
| `df_apps_clean = df_apps_clean[df_apps_clean["Price"] < 250]` | analysis notebook | Removes extreme junk-priced apps from base dataset |
| `Price <= 29.99` | Improvement 2 | Paid viability filter — principled ceiling to remove data-entry junk |
| `min 10 paid apps` | Improvement 2 | Credibility floor — excludes categories with insufficient paid data |
| `KMeans(n_clusters=4, random_state=42)` | Improvement 3 | Cluster count and seed for reproducibility |
| `StandardScaler` | Improvement 3 | Required — prevents install count dominating all other features |
| `fig.write_image('plots/name.png', scale=2)` | all charts | Saves PNG at 150 dpi via kaleido |

---

## 8. Background

This project analyses Google Play Store data originally scraped by Lavanya Gupta in 2018 (available on Kaggle). The dataset was used as the basis for a Plotly data visualisation module covering pie charts, bar charts, scatter plots, and box plots, extended here with market segmentation analysis using scikit-learn.

See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for full concept notes.

---

## 9. Dependencies

| Module | Purpose |
|---|---|
| pandas | Data loading, cleaning, groupby, merge, type conversion |
| numpy | Log transforms, numeric operations |
| plotly | Interactive pie, bar, scatter, and box charts |
| scikit-learn | KMeans clustering, StandardScaler, PCA |
| kaleido | Static PNG export for Plotly charts |
| notebook | Jupyter notebook runtime |

Install all dependencies:

```bash
pip install -r requirements.txt
```
