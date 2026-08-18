# Dataset

## Dataset Overview

- **File:** `games.csv`
- **What one row represents:** A single video game title released on a single
  platform. The same game released on multiple platforms appears as multiple
  rows — 5,155 rows share a `Name` value with at least one other row for this
  reason (verified directly on the file; these are legitimate multi-platform
  releases, not duplicate records — no fully duplicate rows exist in the
  file).
- **Time period covered:** 1980–2016 (`Year_of_Release`)
- **Size:** 16,715 rows × 11 columns
- **Platforms:** 31 distinct platforms
- **Genres:** 12 distinct genres
- **ESRB ratings:** 8 distinct rating categories

This is the exact file used by the analysis notebook
(`notebooks/games_project_Ədalət_Sadıqov.ipynb`, loaded via
`pd.read_csv("../dataset/games.csv")`). It is the **raw** dataset, prior to
the cleaning steps the notebook performs in its Step 2 — see
[Data Cleaning](#data-cleaning-performed-in-the-notebook) below for exactly
what changes at that point.

## Column Dictionary

| Column             | Description                                                                                   | Data Type (raw CSV) | Observed Range / Values |
|--------------------|-------------------------------------------------------------------------------------------------|----------------------|--------------------------|
| `Name`             | Title of the video game.                                                                        | string               | 11,559 unique titles     |
| `Platform`         | Hardware platform the game was released on.                                                     | string               | 31 values — see [Platforms](#platforms) below |
| `Year_of_Release`  | Calendar year the game was released.                                                             | float64              | 1980–2016                |
| `Genre`            | Game genre.                                                                                      | string               | 12 values — see [Genres](#genres) below |
| `NA_sales`         | Sales in North America, in millions of copies.                                                   | float64              | 0.00–41.36                |
| `EU_sales`         | Sales in Europe, in millions of copies.                                                          | float64              | 0.00–28.96                |
| `JP_sales`         | Sales in Japan, in millions of copies.                                                           | float64              | 0.00–10.22                |
| `Other_sales`      | Sales in the rest of the world, in millions of copies.                                           | float64              | 0.00–10.57                |
| `Critic_Score`     | Aggregated professional critic score.                                                            | float64              | 13–98 (scale of 100)      |
| `User_Score`       | Aggregated user/player score. Contains the literal string `"tbd"` (2,424 rows) for titles that have not yet accumulated enough ratings. | string | `"tbd"`, or numeric 0.0–9.7 (scale of 10) |
| `Rating`           | ESRB content rating.                                                                              | string               | 8 values — see [ESRB Ratings](#esrb-ratings) below |

Note: `Critic_Score` and `User_Score` are on **different scales** (0–100 vs.
0–10) — the notebook does not rescale them to compare directly, but analyzes
each against sales independently (see `charts/README.md`, Charts 12–13).

The notebook lowercases every column name in Step 2.1
(`games.columns = games.columns.str.lower()`) and works with `name`,
`platform`, `year_of_release`, `genre`, `na_sales`, `eu_sales`, `jp_sales`,
`other_sales`, `critic_score`, `user_score`, `rating` from that point on. It
also engineers one additional column in Step 2.4:

| Column         | Description                                                | Data Type |
|----------------|--------------------------------------------------------------|-----------|
| `total_sales`  | `na_sales + eu_sales + jp_sales + other_sales` (engineered)  | float64   |

### Platforms

`2600`, `3DO`, `3DS`, `DC`, `DS`, `GB`, `GBA`, `GC`, `GEN`, `GG`, `N64`,
`NES`, `NG`, `PC`, `PCFX`, `PS`, `PS2`, `PS3`, `PS4`, `PSP`, `PSV`, `SAT`,
`SCD`, `SNES`, `TG16`, `WS`, `Wii`, `WiiU`, `X360`, `XB`, `XOne`

### Genres

`Action`, `Adventure`, `Fighting`, `Misc`, `Platform`, `Puzzle`, `Racing`,
`Role-Playing`, `Shooter`, `Simulation`, `Sports`, `Strategy`

### ESRB Ratings

| Rating | Meaning                                   |
|--------|--------------------------------------------|
| `EC`   | Early Childhood                             |
| `E`    | Everyone                                    |
| `E10+` | Everyone 10 and older                       |
| `T`    | Teen                                        |
| `M`    | Mature 17+                                  |
| `AO`   | Adults Only 18+                             |
| `K-A`  | Kids to Adults (legacy rating, pre-1998, later replaced by `E`) |
| `RP`   | Rating Pending                              |

The notebook's regional rating analysis (`charts/README.md`, Chart 18) works
with `games_relevant` (2013–2016 only), where the sales-relevant ratings that
actually appear are `E`, `E10+`, `T`, and `M`.

## Missing Values

Exact null counts on the raw 16,715-row file:

| Column           | Null count | Null % |
|------------------|-----------:|-------:|
| `Name`           | 2          | ~0.0%  |
| `Genre`          | 2          | ~0.0%  |
| `Year_of_Release`| 269        | 1.6%   |
| `Critic_Score`   | 8,578      | 51.3%  |
| `User_Score`     | 6,701      | 40.1%  |
| `Rating`         | 6,766      | 40.5%  |
| `Platform`, `NA_sales`, `EU_sales`, `JP_sales`, `Other_sales` | 0 | 0% |

After the notebook's cleaning steps (see below), on the resulting
16,444-row dataset, the three review-related columns show:

| Column         | Null count (cleaned, n=16,444) | Null % |
|----------------|--------------------------------:|-------:|
| `user_score`   | 8,981                           | 54.6%  |
| `critic_score` | 8,461                           | 51.5%  |
| `rating`       | 6,676                           | 40.6%  |

**6,580 rows have `user_score`, `critic_score`, AND `rating` all missing
simultaneously** — about 40% of the cleaned dataset. The notebook also
checked this pattern by era: among the 1,974 games released between
1980–2000, only 96 have a `critic_score`, 92 have a `user_score`, and 105
have a `rating` — review data is essentially absent for older titles, since
aggregated review-score tracking (e.g. Metacritic) did not exist for most of
that period.

## Special Values

`User_Score` contains the literal string `"tbd"` ("to be determined") for
**2,424 rows** — games that have not yet received an aggregated user score.

**Handling decision (documented in the notebook):** `"tbd"` was converted to
`NaN`, **not** `0`. Treating it as `0` would incorrectly represent an
unrated game as having received the *worst possible* score, which is not
what `"tbd"` means.

## Data Cleaning (performed in the notebook)

Cleaning steps actually applied to this raw file, in order, inside
`notebooks/games_project_Ədalət_Sadıqov.ipynb`:

1. **Column names lowercased.**
2. **Rows with missing `Name` or `Genre` dropped** (2 rows) — these are core
   identifying fields; the remaining data in those rows was judged not
   usable without them. Result: 16,715 → 16,713 rows.
3. **`year_of_release` cast to `Int64`** (pandas' nullable integer type) —
   release years cannot be fractional, but the column still needed to
   support missing values at that point in the pipeline.
4. **`user_score` cleaned:** `"tbd"` replaced with `NaN`, column cast from
   string to `float64`.
5. **Rows with missing `year_of_release` dropped** (269 rows) — judged
   preferable to imputing a release year. Result: 16,713 → 16,444 rows.
   **This 16,444-row cleaned dataset is what nearly all subsequent analysis
   in the notebook uses.**
6. **`critic_score`, `user_score`, and `rating` missing values deliberately
   NOT imputed or dropped.** With 40–54% missingness in each of these three
   columns, and 6,580 rows missing all three simultaneously, the notebook
   concluded that imputation would be unreliable and chose to preserve the
   missing values as-is, only excluding them from specific calculations that
   require them (e.g. `.dropna(subset=['critic_score', 'total_sales'])`
   before computing a correlation).
7. **`total_sales` engineered** as the row-wise sum of `na_sales`,
   `eu_sales`, `jp_sales`, and `other_sales`. Total sales across the entire
   cleaned dataset: **8,814.37 million USD**.
8. **Duplicate check:** confirmed zero fully duplicate rows
   (`games.duplicated().sum()` = 0). Duplicate `Name` values (5,155 rows)
   were investigated and confirmed to be legitimate multi-platform releases
   (e.g. the same title on both PS4 and XOne), not data-entry duplicates —
   no rows were removed on this basis.

A further, analysis-specific filter is applied starting in the notebook's
Step 3.3: most platform/genre/regional/hypothesis-test analysis is
restricted to **`year_of_release` in {2013, 2014, 2015, 2016}** (the
`games_relevant` DataFrame), reasoning that 2013 marked the PS4/XOne
generation launch and that older, discontinued platforms would distort a
2017 forecast. This filter is documented per-chart in `charts/README.md` and
is **not** a permanent modification of the cleaned dataset — `games` (all
years) and `games_relevant` (2013–2016 only) are both used at different
points in the notebook.

## Important Limitations

Limitations identified directly in the notebook:

- **Review-score coverage is heavily skewed toward newer titles.** Critic
  score, user score, and rating are missing for 40–54% of the cleaned
  dataset overall, and for the vast majority of titles released before 2000.
  Any analysis involving these columns implicitly analyzes a
  newer/better-reviewed subset of the market, not the full catalog.
- **2016 data is likely incomplete.** The drop-off in release counts and
  sales after 2012–2013 may partly reflect the dataset's collection cutoff
  rather than an actual decline in the industry, and the notebook explicitly
  flags this when interpreting the release-count trend.
- **The "discontinued platform" classification is dataset-bound.** A
  platform is labeled discontinued if its last recorded release year is more
  than 3 years before the dataset's own maximum year (2016). This can only
  identify platforms that were *already* inactive by 2016 — it cannot
  confirm which platforms remained active after the dataset ends.
- **Digital and mobile sales are likely underrepresented.** The notebook
  notes this as a possible contributor to the apparent post-2012 decline in
  both release counts and sales, since the sales columns appear to track
  traditional retail/physical sales channels.
- **Correlation findings (critic/user score vs. sales) are associative, not
  causal.** The notebook explicitly flags this at every point where a
  correlation is reported (see `charts/README.md`, Charts 12–13).

