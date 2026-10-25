# AEDS6400_blog7
# Exploring Income, Prices, and Spending Across U.S. States

A reproducible R workflow for preparing state-level economic data, building pivot tables, and creating interactive visualizations with Plotly.

This project accompanies Blog Post 7 for *Computational Methods for Economists*.

## 1. Project Purpose

Comparing states using a single indicator can be misleading. High consumption expenditure may accompany high income, high prices, or differences in spending composition.

This project explores three related questions:

1. How does consumption per person differ across states and spending categories?
2. How are personal income and regional price levels associated?
3. How has nominal consumption per person changed over time in selected states?

The main methodological goal is to demonstrate a workflow that begins with official downloadable data and ends with interactive charts embedded in a Quarto blog.

The project provides practice in:

- Reading CSV files directly from ZIP archives.
- Cleaning geographic identifiers.
- Converting wide data into tidy long data.
- Joining tables using geographic and temporal keys.
- Constructing indicators and rankings.
- Creating pivot tables.
- Validating analytical datasets.
- Building interactive charts.
- Exporting and publishing HTML visualizations.

The analysis is descriptive. It does not estimate causal relationships.

## 2. Data Sources

All economic data come from the U.S. Bureau of Economic Analysis (BEA).

Official download portal:

https://apps.bea.gov/regional/downloadzip.htm

### Required datasets

| ZIP archive | BEA table | Information used |
|---|---|---|
| `SAPCE.zip` | `SAPCE2` | Per capita personal consumption expenditures by major type of product |
| `SAINC.zip` | `SAINC1` | Personal income, population, and per capita personal income |
| `SARPP.zip` | `SARPP` | Regional price parities by state |

Direct downloads:

- [Consumption data: SAPCE.zip](https://apps.bea.gov/regional/zip/SAPCE.zip)
- [Income data: SAINC.zip](https://apps.bea.gov/regional/zip/SAINC.zip)
- [Regional price data: SARPP.zip](https://apps.bea.gov/regional/zip/SARPP.zip)

### Files used in this version

- `SAPCE2__ALL_AREAS_1997_2025.csv`
- `SAINC1__ALL_AREAS_1929_2025.csv`
- `SARPP_STATE_2008_2024.csv`

The R code identifies files using filename patterns rather than requiring these exact ending years. This allows it to accommodate some subsequent BEA filename updates.

However, future data releases may also change table structures or definitions, so updated files should always be checked.

### Analytical coverage

- Geography: 50 U.S. states.
- Washington, D.C., national totals, and aggregate regions are excluded from the state-level analytical sample.
- Period: 2008–2024.
- Frequency: Annual.
- State-year observations: 850.

This period provides common coverage across the three datasets.

BEA data can be revised. Downloading the current archives in the future may produce slightly different values. For exact numerical reproduction, retain the original ZIP archives used for the analysis and record the download date.

## 3. Requirements

The analysis uses R and the following packages:

```r
install.packages(c(
  "readr",
  "dplyr",
  "tidyr",
  "stringr",
  "plotly",
  "htmlwidgets"
))
```

Load them before running the analysis:

```r
library(readr)
library(dplyr)
library(tidyr)
library(stringr)
library(plotly)
library(htmlwidgets)
```

RStudio is convenient for viewing tables and charts but is not required for the analytical calculations.

Quarto is required to render `blog7.qmd` into the final blog webpage.

## 4. Files and Organization

The relevant files are:

- `blog7.R`: Data preparation, validation, analysis, and visualization code.
- `blog7.qmd`: Quarto source for the blog.
- `README.md`: Reproduction instructions.
- The three original BEA ZIP archives.
- `output/`: Generated data tables, interactive charts, and validation report.

The blog expects `output/` to be in the same directory as `blog7.qmd`.

The raw ZIP archives can be stored elsewhere because their locations are supplied explicitly in the R script.

This workflow does not require creating a new RStudio Project or changing an existing repository structure.

## 5. How to Run the Analysis

### Step 1: Download the data

Download the three ZIP archives listed above.

Do not manually extract them. The script reads the required CSV files directly from the archives.

### Step 2: Edit local paths

Open `blog7.R` and supply the paths to your downloaded archives:

```r
pce_zip <- "/your/path/SAPCE.zip"
income_zip <- "/your/path/SAINC.zip"
price_zip <- "/your/path/SARPP.zip"
```

For Windows, forward slashes can be used:

```r
pce_zip <- "C:/Users/your_name/Downloads/SAPCE.zip"
```

Check that all three files exist:

```r
file.exists(pce_zip)
file.exists(income_zip)
file.exists(price_zip)
```

Each should return `TRUE`.

### Step 3: Check the working directory

Relative output paths are interpreted from the current working directory.

```r
getwd()
```

Use the directory in which you want the `output/` folder to be created. To use the blog’s existing relative links without modification, this should be the directory containing `blog7.qmd`.

Create the output directory if necessary:

```r
dir.create("output", showWarnings = FALSE)
```

### Step 4: Run the script in order

Run `blog7.R` from beginning to end, or execute its numbered sections sequentially.

Later objects depend on earlier preparation steps. After changing a cleaning rule, rerun all downstream transformations, joins, and validation checks.

### Step 5: Review validation and charts

Confirm that the script reports:

```text
Validation passed.
Number of states: 50
Years: 2008 - 2024
State-year observations: 850
```

Display each chart separately:

```r
fig1
fig2
fig3
```

Then review the exported files in `output/`.

## 6. Data Preparation Workflow

### 6.1 Read the required CSV from each ZIP

The reading function:

1. Lists archive contents using `unzip(..., list = TRUE)`.
2. Identifies the required CSV using a filename pattern.
3. Opens the file using `unz()`.
4. Reads it with `read.csv()`.

The reader preserves original column names with:

```r
check.names = FALSE
```

This is necessary because year columns must remain names such as `2008`, rather than being changed to `X2008`.

Columns are initially read as character values. Numeric conversion is performed explicitly later.

### 6.2 Clean state names

Some income-table state names contain footnote markers, including:

- `Alaska *`
- `Hawaii *`

These names will not match R’s built-in `state.name` vector until cleaned.

The workflow removes asterisks and surrounding whitespace before selecting states. This preserves the observations while standardizing their names.

Without this step, Alaska and Hawaii would be incorrectly excluded from the income and population tables.

### 6.3 Standardize geographic identifiers

BEA geographic codes can contain spaces and quotation marks.

The script removes nonnumeric characters and preserves codes as five-character strings, including leading zeros.

For example, Alabama is represented as `01000`.

Geographic identifiers are used as join keys rather than relying only on state names.

### 6.4 Reshape wide tables into long tables

The original files contain a separate column for each year.

`pivot_longer()` converts those columns into:

- `year`
- `value`

Each observation then represents a particular state, indicator, and year.

This format supports filtering, grouping, joins, and Plotly charts.

### 6.5 Convert numeric values

The script converts:

- Years to integers.
- Indicator codes to integers.
- Indicator values to numeric values using `parse_number()`.

Non-numeric missing-value or suppression markers become missing values and are checked during validation.

### 6.6 Restrict the sample

The script retains:

- States included in `state.name`.
- Years from 2008 through 2024.

National totals and regional aggregate rows are excluded before state-level calculations.

## 7. Indicator Selection and Construction

### Consumption categories

The project selects the following SAPCE2 line codes:

| Line code | Category |
|---:|---|
| 1 | Total consumption |
| 4 | Motor vehicles and parts |
| 9 | Groceries and beverages |
| 10 | Clothing and footwear |
| 11 | Gasoline and energy goods |
| 15 | Housing and utilities |
| 16 | Health care |
| 17 | Transportation services |
| 18 | Recreation services |
| 19 | Food services and accommodations |
| 20 | Financial services and insurance |

Some labels are shortened for chart readability. The original BEA descriptions remain available in the preparation workflow.

These selected categories do not exhaust total consumption. Their shares should not be expected to sum to 100%.

BEA tables contain hierarchical categories. Parent categories and their components must not be added together because that would double-count expenditure.

### Consumption shares

For each state, year, and category:

```text
Category share (%) =
100 × category consumption per person / total consumption per person
```

The denominator is total PCE, not income.

A category’s share measures its contribution to consumption expenditure. It is not a direct measure of affordability or household financial burden.

### Income and population

From SAINC1, the analysis selects:

- Line 2: Population.
- Line 3: Per capita personal income.

Personal income per person is a mean measure, not median household income or an estimate of a particular worker’s salary.

### Regional price parities

SARPP line 1 provides the overall regional price parity.

An RPP of:

- `100` indicates the national price level for that year.
- `110` indicates prices approximately 10% above that level.
- `90` indicates prices approximately 10% below that level.

RPP describes relative price levels across places. It is not a measure of annual inflation.

### Price-adjusted income

The analytical table includes:

```text
Price-adjusted income =
nominal income per person / (RPP / 100)
```

This supports comparisons after adjusting for regional price differences within a year.

It is not a constant-dollar time series and should not be treated as an inflation-adjusted measure across years.

### Growth rates

Annual growth is calculated as:

```text
Growth (%) = 100 × (current value / previous value − 1)
```

The first year for each state has no growth rate because the preceding year is outside the prepared sample.

### Rankings

Income rankings are computed separately within each year.

Rank improvement is defined as:

```text
Nominal income rank − price-adjusted income rank
```

A positive value means the state moves closer to rank 1 after regional price adjustment.

## 8. Joining the Tables

The project builds a state-year analytical table by joining:

1. Total consumption per person.
2. Personal income per person.
3. Population.
4. Overall RPP.

The join keys are:

```r
c("state_fips", "year")
```

The final `state_year_analysis.csv` contains one row per state and year.

The category-level table, `consumption_long.csv`, retains one row per state, year, and selected consumption category.

## 9. Validation Checks

The script checks:

- Exactly 850 state-year observations.
- Unique state-year keys in the analytical table.
- No missing income, total consumption, population, or RPP values.
- Positive RPP values.
- The expected number of category-level observations.
- No missing selected consumption values.

With 11 indicators, 50 states, and 17 years, the consumption table contains:

```text
11 × 50 × 17 = 9,350 rows
```

These checks detect structural problems but do not replace reviewing definitions, units, or source documentation.

Do not remove a failing validation condition simply to continue execution. Investigate the underlying issue first.

## 10. Pivot Tables

### Spending pivot

`pivot_spending.csv` has:

- Rows: States.
- Columns: Consumption categories.
- Values: Annual dollars per person.
- Filter: Selected year, initially 2024.

### Share pivot

`pivot_shares.csv` has:

- Rows: States.
- Columns: Selected categories excluding total consumption.
- Values: Category shares of total PCE.

### Regional pivot

`pivot_region.csv` has:

- Rows: Regions.
- Columns: Years.
- Values: Population-weighted consumption per person.

The regions come from R’s built-in `state.region` classification. Its `North Central` label is retained in the output.

Regional averages use population weights:

```r
weighted.mean(total_pce_per_capita, w = population)
```

A simple mean across states would give small and large states equal weight and answer a different question.

## 11. Interactive Visualizations

### Figure 1: Category Rankings

File:

`figure1_rankings.html`

The chart ranks all 50 states for the selected year.

A dropdown replaces the data in a single bar-chart trace. The update changes:

- Bar lengths.
- State ordering.
- Hover information.
- Chart title.

Hover information includes spending amounts and category shares.

### Figure 2: Income and Prices

File:

`figure2_income_prices.html`

The scatterplot uses:

- Horizontal axis: Personal income per person.
- Vertical axis: Overall RPP.
- Color: Region.
- Year slider: 2008–2024.
- Hover information: State-specific indicators.

Axis ranges remain fixed across years to make comparisons easier.

The horizontal reference line at RPP = 100 identifies the national price level.

### Figure 3: Consumption Trends

File:

`figure3_trends.html`

The chart compares six selected states:

- Pennsylvania.
- New York.
- California.
- Texas.
- Massachusetts.
- Florida.

Readers can toggle states through the legend and select a shorter interval using the range slider.

The values are in current dollars. Trends therefore reflect price changes as well as changes in quantities and expenditure composition.

## 12. Generated Outputs

| File | Contents |
|---|---|
| `consumption_long.csv` | Category-level tidy consumption data |
| `state_year_analysis.csv` | Joined state-year analytical dataset |
| `pivot_spending.csv` | Spending amount pivot |
| `pivot_shares.csv` | Consumption share pivot |
| `pivot_region.csv` | Population-weighted regional pivot |
| `check_results.txt` | Validation summary and descriptive results |
| `figure1_rankings.html` | Interactive category rankings |
| `figure2_income_prices.html` | Interactive income-price scatterplot |
| `figure3_trends.html` | Interactive consumption trends |

The HTML exports use:

```r
selfcontained = FALSE
```

Each HTML file therefore requires its associated `_files` directory. Preserve both when moving or publishing charts.

## 13. Rendering the Quarto Blog

The blog embeds exported charts using HTML iframes.

For example:

```html
<iframe
  src="output/figure2_income_prices.html"
  title="State income and regional prices"
  width="100%"
  height="650"
  style="border: none;">
</iframe>
```

This approach preserves interactivity without rerunning the R analysis during blog rendering.

Render the blog using the RStudio Render button or:

```bash
quarto render blog7.qmd
```

If an existing Quarto project uses a separate publishing directory, ensure the chart resources are copied there. The supplied QMD includes resource declarations for `output/**` and `blog7.R`.

After rendering, verify all three charts in a browser.

When publishing, preserve the relative paths between the blog page, chart HTML files, and dependency folders.

## 14. Troubleshooting

### Missing Alaska or Hawaii

Clean footnote asterisks from geographic names, then rerun transformations, joins, and validation.

### ZIP file not found

Check the full local path with `file.exists()`. The Files pane and current working directory may point to different locations.

### Export directory not found

Run:

```r
dir.create("output", showWarnings = FALSE)
```

### Chart is blank

Test a minimal Plotly chart and open the exported chart in an independent browser to distinguish preview problems from chart construction problems.

Figure 1 uses a single trace with dropdown-driven data replacement.

### Chart breaks after moving files

Check that the HTML file and its corresponding `_files` directory were moved together.

### RStudio shows `Browse[1]>`

R is paused in a debugging environment. Exit with uppercase `Q`, then rerun the required definitions and reading steps in the normal console.

### Width or height deprecation warning

Specify chart dimensions in `plot_ly()` rather than `layout()`.

## 15. Interpretation Limits

- State averages conceal differences within states.
- Personal income is not median household income or wages.
- PCE includes expenditure made on residents’ behalf and is not limited to direct household payments.
- Current-dollar growth should not be interpreted as real consumption growth.
- RPP adjusts regional price differences, not differences in taxes, household needs, or every aspect of living costs.
- State-level correlations do not establish causality or describe individual-level relationships.
- Rankings are sensitive to revisions and can exaggerate small numerical differences.
- Selected consumption categories do not form an exhaustive expenditure decomposition.

## 16. Reproducibility Checklist

Before considering a reproduction complete, confirm that:

- [ ] All three original ZIP archives are available.
- [ ] Local input paths have been updated.
- [ ] Required R packages are installed.
- [ ] Geographic footnote markers have been cleaned.
- [ ] The analytical sample contains 50 states and 17 years.
- [ ] Validation passes.
- [ ] Tables and charts are regenerated from the same inputs.
- [ ] Dropdown, slider, hover, and legend interactions work.
- [ ] Exported HTML files retain their dependency directories.
- [ ] The rendered blog displays all embedded charts.
- [ ] Download date and package versions are recorded.

To record software versions, run:

```r
sessionInfo()
```

Optionally save the information:

```r
writeLines(
  capture.output(sessionInfo()),
  "output/session_info.txt"
)
```

## Author

Yixin Yang  
University of Pennsylvania  
Computational Methods for Economists — Blog Post 7