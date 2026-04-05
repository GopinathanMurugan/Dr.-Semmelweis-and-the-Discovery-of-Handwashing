# Dr. Semmelweis and the Discovery of Handwashing
### Analysing Historical Medical Data to Prove the Life-Saving Impact of Hand Hygiene

## Table of Contents
1. [Project Overview](#project-overview)
2. [Data Source](#data-source)
3. [Tool](#tool)
4. [Data Cleaning](#data-cleaning)
5. [Exploratory Data Analysis](#exploratory-data-analysis)
6. [Data Analysis](#data-analysis)
7. [Results](#results)
8. [Recommendations](#recommendations)
9. [Contact](#contact)

---

## Project Overview

This project recreates the historic data analysis performed by **Dr. Ignaz Semmelweis** — a Hungarian physician (1818–1865) who worked at the Vienna General Hospital — to investigate why so many women were dying from **childbed fever (puerperal fever)** in maternity wards during the 1840s.

By stepping into Dr. Semmelweis's shoes and analysing the same data he collected between **1841 and 1849**, this project demonstrates the power of data-driven decision making in medicine and proves — using statistical analysis — that mandatory handwashing dramatically reduced maternal mortality.

The analysis covers:
- **Clinic Comparison:** Comparing annual birth and death rates between Clinic 1 (staffed by medical students who performed autopsies) and Clinic 2 (staffed by midwives) to identify a significant and unexplained disparity in death rates.
- **Mortality Rate Calculation:** Computing the proportion of women dying during childbirth at Vienna General Hospital in the 1840s and comparing it to modern maternal mortality benchmarks.
- **Handwashing Impact Analysis:** Splitting monthly data into before (pre-June 1847) and after (post-June 1847) periods to quantify the reduction in death rates after Dr. Semmelweis made handwashing with chlorine mandatory.
- **Time Series Visualisation:** Plotting births and deaths over time using twin-axis Matplotlib charts with year/month tick locators to identify the dramatic inflection point in mid-1847.
- **Rolling Average Analysis:** Computing a 6-month rolling average of the death rate to smooth short-term fluctuations and reveal the underlying trend before handwashing.
- **Statistical Significance Testing:** Applying an independent samples t-test to confirm that the reduction in deaths after handwashing is statistically significant and not due to chance.
- **Distribution Comparison:** Using box plots, overlapping histograms, and Kernel Density Estimates (KDE) to visually compare the distribution of monthly death rates before and after handwashing.

---

## Data Source

The data originates from **Dr. Semmelweis's own published research (1861)** and covers the Vienna General Hospital maternity wards from 1841 to 1849.

### Dataset 1 — `annual_deaths_by_clinic.csv`
**Records:** 12 rows | **Grain:** One row per clinic per year

| Column | Description |
|---|---|
| `year` | Year of observation (1841–1846) |
| `births` | Total number of births in that year |
| `deaths` | Total number of maternal deaths in that year |
| `clinic` | Clinic name (`clinic 1` or `clinic 2`) |

**Note:** Clinic 1 was staffed by medical students who routinely performed post-mortem autopsies before delivering babies. Clinic 2 was staffed by midwives only.

---

### Dataset 2 — `monthly_deaths.csv`
**Records:** 98 rows | **Grain:** One row per month (January 1841 to March 1849)

| Column | Type | Description |
|---|---|---|
| `date` | DateTime | First day of the month (parsed as datetime) |
| `births` | Integer | Total births in that month |
| `deaths` | Integer | Total maternal deaths in that month |

**Key Period:** June 1847 is the critical inflection point — the month Dr. Semmelweis made chlorine handwashing mandatory for all medical staff.

---

## Tool

| Tool | Purpose |
|---|---|
| **Python 3** | Core programming language |
| **Pandas** | Data loading, manipulation, derived column creation, date parsing |
| **NumPy** | Numerical operations, `.where()` for conditional column labelling |
| **Matplotlib** | Twin-axis time series charts with year/month tick locators |
| **Plotly Express** | Interactive line charts, box plots, and histograms |
| **Seaborn** | Kernel Density Estimate (KDE) plots with shading and clipping |
| **SciPy (stats)** | Independent samples t-test for statistical significance testing |

---

## Data Cleaning

- **Parse Dates at Import:** The `date` column in `monthly_deaths.csv` is parsed directly at load time using `parse_dates=['date']` in `pd.read_csv()` — avoiding manual `to_datetime()` conversion later and ensuring correct time-series operations.
- **Check for Missing Values:** `.info()` and `.isna().values.any()` confirmed zero NaN values in both datasets — no imputation required.
- **Check for Duplicates:** `.duplicated().values.any()` confirmed no duplicate rows in either dataset — all 12 annual and 98 monthly records are unique.
- **Derive Percentage Death Column:** Added a `pct_deaths` column to both `df_yearly` (`deaths / births`) and `df_monthly` (`deaths / births`) to enable proportional comparisons that account for varying birth volumes across months and clinics.
- **Label Pre/Post Handwashing Period:** Used `numpy.where()` to add a `washing_hands` column to `df_monthly` — assigning `'No'` for dates before June 1847 and `'Yes'` for dates from June 1847 onward — enabling grouped visualisation and statistical comparison.
- **Set Date as Index for Rolling Average:** Converted the `date` column to the DataFrame index using `.set_index('date')` before applying `.rolling(window=6).mean()` — preventing the date column from being inadvertently dropped during the rolling calculation.
- **Clip KDE at Zero:** Applied `clip=(0, 1)` in the Seaborn KDE plot to prevent the default distribution estimate from extending into negative death rate territory — which is mathematically impossible and misleading.

---

## Exploratory Data Analysis

- **Dataset Shape:** `df_yearly` is 12 rows × 4 columns; `df_monthly` is 98 rows × 3 columns — both fully complete with no missing values or duplicates.
- **Clinic Size Comparison:** Plotly line charts of annual births by clinic showed Clinic 1 consistently had more patients than Clinic 2, with both clinics growing in patient volume from 1841 to 1846.
- **Descriptive Statistics:** `.describe()` on both datasets revealed the range of births, deaths, and derived proportions — establishing baseline statistical context before the intervention analysis.
- **Overall Mortality Rate:** The overall maternal death rate at Vienna General Hospital in the 1840s was approximately **9.92%** — compared to just 0.018% in the United States in 2013 — highlighting how catastrophically dangerous childbirth was before germ theory was understood.
- **Clinic Death Rate Disparity:** Average death rate in Clinic 1 was approximately **10.5%**; Clinic 2 was approximately **3.9%** — a stark 2.7× difference that Semmelweis sought to explain.
- **Twin-Axis Time Series:** Matplotlib charts with twin y-axes plotted monthly births (skyblue, linewidth=3) and deaths (crimson dashed, linewidth=2) simultaneously from 1841 to 1849, with year and month tick locators — revealing a dramatic drop in deaths in mid-1847.

---

## Data Analysis

**Clinic Comparison (Annual Data):**
Calculated per-row `pct_deaths` for each clinic-year combination. Plotted separate Plotly line charts for births and deaths by clinic. Computed average death rates: Clinic 1 = ~10.5%, Clinic 2 = ~3.9%. The persistent gap suggested an environmental or procedural cause specific to Clinic 1 — the cadaverous particles hypothesis that led Semmelweis to mandate handwashing.

**Pre/Post Handwashing Split (Monthly Data):**
Set `handwashing_start = pd.to_datetime('1847-06-01')`. Split `df_monthly` into two subsets:
- `before_washing` — all months before June 1847 (72 records)
- `after_washing` — June 1847 onward (26 records)

Computed aggregate death rates for each period and average monthly percentages for both groups.

**6-Month Rolling Average:**
Computed a 6-month rolling mean of `pct_deaths` on the `before_washing` subset (with date as index) to smooth monthly variation and reveal the underlying trend leading up to the handwashing intervention.

**Time Series Visualisation (Monthly Death Rate):**
Built a Matplotlib chart overlaying three lines:
- Thin dashed black line: monthly death rate before handwashing
- Thick crimson dashed line: 6-month moving average before handwashing
- Skyblue line with round markers: monthly death rate after handwashing

Legend added with `plt.legend(handles=[...])`. The chart strikingly shows the death rate falling immediately after June 1847.

**Statistical Analysis:**
- Mean death rate before handwashing: ~**10.5%**
- Mean death rate after handwashing: ~**2.15%**
- Absolute reduction: ~**8.4 percentage points**
- Relative improvement: approximately **5x lower** death rate after handwashing

**Box Plot Comparison:**
Plotly box plot grouped by `washing_hands` ('Yes'/'No') showed the median, IQR, and outliers for both periods — visually confirming that the central tendency, spread, and worst-case death rates all improved dramatically after handwashing.

**Histogram Comparison:**
Plotly overlapping histograms with `histnorm='percent'` (to account for unequal time period lengths), `opacity=0.6`, `nbins=30`, and a marginal box plot — showing the distribution of monthly death rates shifted substantially left after handwashing began.

**KDE Analysis:**
Seaborn KDE plots with `shade=True` and `clip=(0,1)` showed near-zero death rate distribution after handwashing vs. a wide, high-mean distribution before — making the practical impact of the intervention visually intuitive.

**T-Test (Statistical Significance):**
Applied `scipy.stats.ttest_ind(before_washing.pct_deaths, after_washing.pct_deaths)`:
- p-value < 0.0001 (well below the 1% significance threshold)
- t-statistic confirms the means are significantly different

The difference in death rates before and after handwashing is **statistically significant at the 99% confidence level** — meaning there is less than a 1% probability the improvement was due to chance.

---

## Results

- **Overall 1840s Mortality Rate:** Approximately **9.92%** of women who gave birth at Vienna General Hospital in the 1840s died — roughly **550× higher** than the modern US maternal mortality rate of 0.018%.
- **Clinic 1 vs. Clinic 2:** Clinic 1 (medical students with autopsy exposure) had a death rate of ~10.5% vs. ~3.9% in Clinic 2 (midwives only) — a consistently higher mortality rate driven by cadaverous contamination before handwashing practices existed.
- **Handwashing Reduction:** Mandatory chlorine handwashing reduced the average monthly death rate from approximately **10.5% to 2.15%** — an absolute reduction of over 8 percentage points and approximately a **5× improvement** in survival odds.
- **Statistical Significance:** The independent samples t-test returned a p-value of less than 0.0001 — confirming with 99%+ confidence that handwashing caused the reduction in deaths, not random variation.
- **Distribution Shift:** Both the box plots and KDE plots showed the entire distribution of monthly death rates shifted dramatically left after June 1847, with both the mean and worst-case months improving substantially.
- **Rolling Average Confirmation:** The 6-month rolling average showed a steadily elevated and volatile death rate throughout 1841–1847, which immediately stabilised at a much lower level after the handwashing mandate — ruling out seasonal effects.

---

## Recommendations

- **Use Proportional Rates for Fair Comparison:** Always compare death rates as a percentage of births rather than raw death counts — Clinic 1 had more patients, so raw numbers alone would be misleading. This principle applies broadly to any healthcare or population-level data analysis.
- **Split Time Series at Intervention Points:** When analysing policy or treatment interventions, always split the time series at the exact intervention date and compute separate statistics for each period — as done here with June 1847 — to isolate the causal effect.
- **Apply Rolling Averages to Reveal Trends:** A 6-month rolling average smooths short-term noise (seasonal variation, reporting anomalies) and makes the underlying trend more visible — especially useful for healthcare data with high month-to-month variability.
- **Use T-Tests to Validate Visual Patterns:** Visual differences in charts are compelling but not sufficient — always support observations with a formal statistical test (e.g., `scipy.stats.ttest_ind`) to confirm that observed differences are statistically significant and not due to chance.
- **Combine Multiple Chart Types for Robust Insight:** Line charts reveal trends over time; box plots reveal distributional statistics; histograms reveal frequency distributions; KDE plots reveal smooth probability shapes. Using all four together (as in this project) gives a more complete picture than any single visualisation.
- **Clip KDE Plots at Domain Boundaries:** Always apply `clip=(0, 1)` or appropriate bounds in Seaborn KDE plots when the variable has natural limits — unconstrained KDE can suggest impossible values (e.g., negative death rates) that mislead interpretation.
- **Parse Dates at Import:** Use `parse_dates=['date_column']` in `pd.read_csv()` rather than converting after the fact — it prevents dtype warnings, enables immediate time-series operations, and avoids subtle bugs from string-formatted dates.

---

## Contact

For any questions or further information, please contact: **gopinathan.stat@gmail.com**

Feel free to modify any section or add additional information as needed!
