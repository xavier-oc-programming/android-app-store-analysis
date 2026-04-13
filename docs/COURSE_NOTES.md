# Course Notes — Day 76: Beautiful Plotly Charts and Analysing the Android App Store

## Exercise Brief

Analyse Google Play Store data and turn raw app metrics into clear, insightful, and visually appealing charts using Plotly.

The goal is to understand how apps perform across categories, pricing models, and popularity, replicating real-world analytics used by companies such as App Annie and Sensor Tower to guide product and marketing decisions.

---

## Key Questions

- How competitive are different app categories (Games, Lifestyle, Weather, etc.)?
- Which app categories offer the strongest download opportunities?
- How many downloads are lost when switching from free to paid?
- How much can you reasonably charge for a paid app?
- Which paid apps generate the highest estimated revenue?
- How many paid apps recover their development costs through sales?

---

## Key Concepts Covered

### Data Cleaning
- `.drop()` — remove unused columns (`Last_Updated`, `Android_Ver`)
- `.isna()` / `.dropna()` — identify and remove NaN rows
- `.duplicated()` / `.drop_duplicates(subset=[...])` — detect and remove duplicates correctly by specifying a meaningful subset

### Numeric Type Conversion
- `str.replace()` — strip commas from Installs, dollar signs from Price
- `pd.to_numeric()` — convert cleaned strings to numeric dtype

### Exploration
- `.sort_values()` — rank apps by Rating, Size, Reviews
- `.groupby()` — count apps and sum installs per category
- `.merge()` — join two grouped DataFrames on Category

### Nested / Compound Data
- `.str.split(';', expand=True).stack()` — flatten semi-colon-separated Genres into one Series
- `.explode()` — alternative approach to flatten list columns
- `.value_counts()` — count genre frequency after flattening

### Aggregation
- `.agg({'App': pd.Series.count})` — count apps per group
- `.groupby(["Category", "Type"])` — multi-key groupby for free vs paid breakdown

### Plotly Visualisations
- `px.pie()` — content rating distribution (pie and donut)
- `px.bar()` — top genres by count with colour scale; free vs paid apps per category (grouped)
- `px.scatter()` — category concentration (number of apps vs installs), bubble size by count
- `px.box()` — installs for free vs paid; revenue estimates by category; price by category

---

## Dataset Notes

- Scraped from the Google Play Store by Lavanya Gupta in 2018
- Source: https://www.kaggle.com/lava18/google-play-store-apps
- 10,841 rows × 12 columns before cleaning; 8,199 rows after
- Prices in USD at time of scraping
- Installs are rounded magnitude ranges (e.g. `100,000+`), treated as exact after stripping symbols
- App size in MB; rows with missing size not removed (Rating NaNs removed instead)
