# Video Game Sales Analysis

## Project Overview

This project analyzes a historical dataset of video game sales (1980–2016)
covering 16,715 titles across 31 platforms and 12 genres, with regional sales
figures for North America, Europe, Japan, and the rest of the world, plus
critic scores, user scores, and ESRB content ratings where available.

The analysis was performed to identify which platforms, genres, and regions
are currently driving sales, understand how the console market has shifted
generations over time, and produce evidence-based advertising recommendations
for the 2017 planning period. The dataset was cleaned, explored, and modeled
entirely in Python (pandas, NumPy, SciPy), with all visualizations built in
Plotly.

## Research Questions

The notebook addresses the following questions:

- How have video game release volumes changed from 1980 to 2016?
- How long does a gaming platform typically stay commercially active, and
  which platforms had already become discontinued by 2016?
- Which time period is most representative of the *current* console market
  for forecasting purposes, rather than the full 36-year history?
- Which platforms generated the most sales in that relevant period, and which
  are growing vs. declining?
- Does a game's critic score or user score correlate with its sales (using
  PS4 as a case study)?
- Which multi-platform games sell best, and how does their performance split
  across platforms?
- Which genres dominate the market, by both total sales and market share?
- How do platform, genre, and ESRB rating preferences differ across North
  America, Europe, and Japan?
- Is there a statistically significant difference in user scores between the
  Xbox One and PC platforms?
- Is there a statistically significant difference in user scores between the
  Action and Sports genres?

## Dataset

- 16,715 rows × 11 raw columns; 16,444 rows after cleaning
- Core fields: `name`, `platform`, `year_of_release`, `genre`
- Regional sales: `na_sales`, `eu_sales`, `jp_sales`, `other_sales` (millions
  of copies), plus an engineered `total_sales` column
- Review fields: `critic_score` (0–100), `user_score` (0–10), `rating` (ESRB) —
  each 40–54% missing, concentrated in older titles
- Time period: 1980–2016

Full column-by-column documentation, missing-value analysis, and cleaning
decisions are in **[`dataset/README.md`](dataset/README.md)**.

## Data Cleaning and Preparation

- Standardized all column names to lowercase.
- Dropped 2 rows missing `name`/`genre` (core identifying fields).
- Converted `year_of_release` to a nullable integer type.
- Converted `"tbd"` values in `user_score` to `NaN` (not `0`, to avoid
  implying the worst possible score for an unrated game), then cast the
  column to `float64`.
- Dropped 269 rows missing `year_of_release` (16,713 → 16,444 rows).
- Deliberately **kept** missing values in `critic_score`, `user_score`, and
  `rating` rather than imputing them — with 40–54% missingness in each
  column and 6,580 rows missing all three at once, the notebook judged
  imputation to be unreliable at that scale.
- Engineered `total_sales = na_sales + eu_sales + jp_sales + other_sales`.
- Confirmed no fully duplicate rows exist in the dataset.

## Exploratory Data Analysis

- **Temporal analysis:** annual release counts, 1980–2016.
- **Platform lifecycle:** first/last active year and lifespan per platform,
  discontinued-platform classification, sales heatmap across
  platform × year.
- **Relevant-period selection:** restricting most downstream analysis to
  2013–2016 based on the PS4/XOne generation launch.
- **Platform performance:** total sales, year-over-year growth, and a linear
  2017 sales forecast per platform.
- **Sales distribution:** per-platform sales spread (median, mean, std),
  both all-time and for 2013–2016.
- **Review-score impact:** critic score and user score vs. sales, using PS4
  as a case study.
- **Cross-platform comparison:** how the same multi-platform game performs
  across the platforms it was released on.
- **Genre analysis:** total sales and market share by genre.
- **Regional analysis:** platform, genre, and ESRB rating preferences broken
  out by North America, Europe, and Japan.

## Statistical Analysis

Two independent-samples t-tests (Welch's t-test, unequal variances) were
performed, both at **α = 0.05**.

### Test 1 — Xbox One vs. PC user scores

- **H0:** No statistically significant difference between mean `user_score`
  on Xbox One and PC (μ_XOne = μ_PC).
- **H1:** A statistically significant difference exists (μ_XOne ≠ μ_PC).
- **Test:** Welch's t-test (`scipy.stats.ttest_ind`, `equal_var=False`)
- **Sample sizes:** Xbox One n=182 (mean=6.521, std=1.381); PC n=155
  (mean=6.270, std=1.742)
- **Result:** t = 1.452, p = 0.1476
- **Decision:** p ≥ 0.05 → **H0 is not rejected.**
- **Interpretation:** No statistically significant difference was detected
  between Xbox One and PC user scores in this dataset.

### Test 2 — Action vs. Sports user scores

- **H0:** No statistically significant difference between mean `user_score`
  for Action and Sports genres.
- **H1:** A statistically significant difference exists.
- **Test:** Welch's t-test (`scipy.stats.ttest_ind`, `equal_var=False`)
- **Sample sizes:** Action n=389 (mean=6.838, std=1.330); Sports n=160
  (mean=5.238, std=1.783)
- **Result:** t = 10.233, p ≈ 1.45×10⁻²⁰
- **Decision:** p < 0.05 → **H0 is strongly rejected.**
- **Interpretation:** Action games are rated significantly higher by users
  than Sports games on average. The notebook offers one possible (untested)
  explanation — Sports titles often follow annual roster-update cycles with
  less year-to-year innovation than Action titles — while explicitly noting
  this is an interpretation, not proven causation.

## Key Findings

1. **Platform lifecycle:** Platforms stay active for a median of 6 years
   (mean 7.6 years). Using a 3-year inactivity threshold, 20 platforms
   (PS2, XB, GBA, GC, PS, N64, SNES, and others) are classified as
   discontinued by 2016. The 2013 PS4/XOne launch marked a clear generation
   shift, which is why **2013–2016** was selected as the relevant period for
   the 2017 forecast.
2. **Current-generation leaders (2013–2016):** PS4 leads with 314.14M in
   sales, followed by PS3 (181.43M), XOne (159.32M), 3DS (143.25M), and X360
   (136.80M). PS4 and XOne show a growing trend; PS3/X360/Wii/DS are
   declining as older-generation platforms.
3. **Reviews (PS4 case study):** `critic_score` has a moderate positive
   correlation with sales (r ≈ 0.41); `user_score` shows almost no
   correlation (r ≈ −0.03). Correlation is not causation — other factors
   like marketing and budget likely play a role.
4. **Genre (all-time, 1980–2016):** `Action` leads with ≈19.5% market share
   (1,716.52M), followed by `Sports` (14.9%), `Shooter` (11.8%), and
   `Role-Playing` (10.6%). `Adventure`, `Strategy`, and `Puzzle` are the
   smallest niches.
5. **Regional differences (2013–2016):**
   - *Platforms:* `PS4` leads in both NA (24.84% share) and EU (35.97%
     share). Japan is dominated by `3DS` (48.17% share) — a clear outlier
     from the Western pattern.
   - *Genres:* NA and EU both favor `Action` > `Shooter` > `Sports`. Japan
     favors `Role-Playing` (36.26% share) over `Action` (28.76%), with
     `Shooter` far behind (4.70%).
   - *ESRB rating:* `M`-rated games lead sales in NA (165.21M) and EU
     (145.32M). Japan is led by `T`-rated games (20.59M) instead.
6. **Hypothesis tests:** No significant difference in user scores between
   XOne and PC (p = 0.148). Highly significant difference between Action and
   Sports user scores (p ≈ 1.45×10⁻²⁰), with Action rated higher on average
   (6.84 vs. 5.24).

### 2017 Advertising Recommendations (from the notebook)

- Focus advertising spend on `PS4` titles, and on the `Action`/`Shooter`
  genres in NA and EU — these show the strongest current sales and align
  with each region's top platform and genre preferences.
- Use a distinct strategy for Japan: prioritize `Role-Playing` titles on
  `3DS`, and lean toward broader-audience (`T`-rated) content rather than
  mature-only titles, since Japan's regional profile diverges sharply from
  NA/EU on every dimension measured.
- Rely more on critic scores than user scores when using reviews as a
  marketing signal, since critic scores show a stronger (though still
  moderate) relationship with sales.

## Visualizations

All 19 charts generated in the notebook are exported as individual PNG files
in **[`charts/`](charts/)**, each with a full explanation of its purpose,
what it shows, and its analytical interpretation in
**[`charts/README.md`](charts/README.md)**.

## Project Structure

```text
video-game-sales-analysis/
│
├── README.md                     # This file
├── requirements.txt               # Python dependencies
├── .gitignore
│
├── notebooks/
│   └── games_project_Ədalət_Sadıqov.ipynb   # Main analysis notebook
│
├── charts/
│   ├── README.md                  # Explanation of every chart
│   └── 01_*.png ... 19_*.png      # Exported Plotly visualizations
│
├── dataset/
│   ├── README.md                  # Full dataset documentation
│   └── games.csv                  # Source dataset (16,715 rows x 11 columns)
│
└── src/
    ├── README.md
    └── extract_charts.py          # Re-exports notebook charts to PNG via Plotly/Kaleido
```

## Technologies Used

- Python
- pandas
- NumPy
- SciPy (`scipy.stats` — t-tests, linear regression)
- Plotly (`plotly.express`, `plotly.graph_objects`) — all visualizations
- Kaleido — static PNG export of Plotly figures
- ipywidgets
- Jupyter Notebook

No Matplotlib or Seaborn is used anywhere in this project.

## How to Run

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd video-game-sales-analysis
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. The dataset is already included at `dataset/games.csv` (see
   [`dataset/README.md`](dataset/README.md) for full schema documentation).
4. Open the notebook:
   ```bash
   jupyter notebook notebooks/games_project_Ədalət_Sadıqov.ipynb
   ```
5. Run all cells from top to bottom.

To regenerate the chart PNGs from the notebook's saved outputs without
re-running the full analysis, see [`src/README.md`](src/README.md).

## Conclusion

Between 2013 and 2016, the video game market underwent a clear generational
transition — PS4 and Xbox One displacing the previous console generation —
which makes this window the most reliable basis for near-term (2017)
forecasting. Within that window, sales are concentrated in a small number of
platforms (led by PS4) and genres (led by Action and Shooter), critic reviews
carry a moderate positive relationship with sales while user reviews do not,
and regional preferences diverge sharply between Western markets and Japan
across platform, genre, and content-rating dimensions alike. These patterns
directly support the region-specific 2017 advertising recommendations
summarized above, while the notebook consistently distinguishes statistically
supported findings from interpretive explanations that were not directly
tested.
