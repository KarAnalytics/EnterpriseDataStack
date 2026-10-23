# Data Profiling

Before you trust a dataset for a business decision, you need to know what it actually contains, not what its documentation says it contains. **Data profiling** examines the data and produces informative summaries of it: how complete each column is, what values occur, what patterns they follow, and where they break the rules. This chapter explains why profiling matters, what to check, how to measure it, and how to profile with tools in Python, R and SQL.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain the purpose of data profiling and its relationship to data processing and data quality.
2. Plan a profile table by table, column by column and rule by rule.
3. Compute and interpret NULL, missing, actual and cardinality counts, and completeness, uniqueness and distinctness.
4. Profile a dataset with fg-data-profiling in Python, DataExplorer in R, and aggregate queries in SQL.
5. Turn profiling findings into processing actions.
:::

## Extract, process, profile

Three activities make data business-ready:

- **Information extraction**: automated retrieval of specific information from text, documents, databases, websites or other sources.
- **Data processing**: adjusting data so it is organized, easy to read and ready for downstream use ([Data Processing](../data-processing/intro.md)).
- **Data profiling**: examining the data available from an existing source and preparing informative summaries about it.

Profiling and processing go hand in hand: profile, process, and profile again, until the data quality is adequate for its purpose.

## Why profile?

- Determine whether existing data can be **repurposed** for a new use.
- Check whether the data **conforms to company standards**.
- Assess the **risk of integrating** the data into new applications.
- **Track data quality** over time.
- Check whether the **metadata** (column names, documented types and meanings) describes the actual values.
- Find **data challenges early**, so there are no surprises late in a project.
- Support **master data management** and **data governance**.

## Data quality

Profiling serves **data quality (DQ)**, which is usually defined as **fitness for use**. Quality depends on the decision: data complete enough for a sales forecast may be far too incomplete for the accounting books, even for the same business user. Data quality determines the value of data to the business. High-quality data combined with good technology is a great asset; poor-quality data combined with good technology is an equally great liability, the **garbage in, garbage out (GIGO)** principle. Decisions made on bad data have cost companies billions of dollars.

Some ways bad data leads to bad decisions:

- **Obsolete values**: roughly 10% of customers change their address each year, so an old customer list quickly misdirects mail, bills and services.
- **Unavailable inputs**: a prediction model that uses a variable which will not be available when the model is actually used.
- **Missing drivers**: an explanatory model that leaves out important factors.
- **Too much or too little data**, or **over-synthesized data** that no longer reflects reality.

## What a profile checks

Profiling analyzes data in several ways:

- Verify that the data **matches the metadata** that describes it.
- Identify how the **absence of data** is represented (blank, NULL, "N/A", 0, 9999, ...).
- Identify potential **default and invalid values**.
- Check **formats** for inconsistencies.
- Assess **domain** (valid values), **structural** and **referential integrity**.
- Assess and **discover business rules**.

### Steps

1. **Plan the profile**: table by table, column by column, and business rule by business rule.
2. **Interpret the results**: what does each finding mean for the intended use?
3. **Process and repeat**: fix what needs fixing, then profile again until the quality is satisfactory.

### A basic checklist

- Check the **data volume** and basic metadata (file type, size, encoding) before opening the file.
- Is the data **structured, semi-structured or unstructured**?
- Are there **duplicate records**?
- Are the **input and outcome variables** identified?
- Do the **data types** match expectations?
- Do the **distributions** of continuous and discrete variables match expectations?
- Are there **missing values, outliers or special observations**?
- Are **timestamps** (date, time) and **spatial** values (latitude, longitude, address) formatted consistently?
- What do **one-, two- and three-way plots** show?

### From diagnosis to action


:::{figure} images/profiling_processing.png
:label: fig-profiling-processing
:alt: Two columns of five boxes linked by double arrows: profiling diagnostics on the left and the matching data processing actions on the right.
Data profiling and data processing go hand in hand: each diagnostic points to a processing action.
:::

| Profiling diagnostic | Processing action |
|---|---|
| Identify input and outcome variables; is the data structured or unstructured? | Exclude irrelevant variables; integrate external sources to enhance the data. |
| Do the actual types and formats match the business rules? | Verify, correct and standardize types, units, locations and timestamps. |
| Scan for missing values, outliers, special observations and duplicates. | Handle missing values, treat outliers, remove duplicates. |
| Do the distributions match expectations? | Discretize or transform numeric variables; group levels and one-hot encode categorical ones. |
| Examine cross-variable relationships, logical dependencies and class imbalance. | Engineer new variables; augment data where classes are imbalanced. |

## Measuring a column

A column profile counts four things:

- **NULL**: the number of NULL (empty) values.
- **Missing**: the number of values that are not NULL but still represent absent data, such as "N/A", "unknown", a single space or a placeholder like 9999.
- **Actual**: the number of real values (neither NULL nor missing).
- **Cardinality**: the number of distinct actual values.

and three ratios:

- **Completeness** = actual / total records. How much of the column is filled?
- **Uniqueness** = cardinality / total records. Could the column identify rows (100%)?
- **Distinctness** = cardinality / actual. Among the filled values, how varied are they?

For the 500 rows of the Fortune Global 500 file from the [previous chapters](../data-processing/intro.md) (it has no "missing" placeholders, so Missing is 0 throughout):

| Column | NULL | Actual | Cardinality | Completeness | Uniqueness | Distinctness |
|---|---|---|---|---|---|---|
| Company | 0 | 500 | 500 | 100.0% | 100.0% | 100.0% |
| HqCountry | 0 | 500 | 35 | 100.0% | 7.0% | 7.0% |
| IndustrySector | 0 | 500 | 57 | 100.0% | 11.4% | 11.4% |
| EmployeeCount | 0 | 500 | 492 | 100.0% | 98.4% | 98.4% |
| RankCng | 55 | 445 | 165 | 89.0% | 33.0% | 37.1% |
| ProfitsPerCng | 37 | 463 | 212 | 92.6% | 42.4% | 45.8% |


:::{figure} images/fortune_profile.png
:label: fig-fortune-profile
:alt: A bar chart of completeness and uniqueness for six Fortune Global 500 columns. Company is 100% complete and unique; HqCountry and IndustrySector are complete but have low uniqueness; RankCng is 89% complete.
Completeness and uniqueness of six columns of the Fortune Global 500 file.
:::

Reading the profile:

- **Company** is 100% unique, so it can serve as an identifier.
- **HqCountry** has low uniqueness, as a category should: 35 countries.
- **IndustrySector** has 57 values, more than a clean industry list should, which prompts a closer look (it mixes two classification schemes; see the [OpenRefine chapter](../openrefine/intro.md)).
- **RankCng** is only 89% complete, and the numbers alone do not say why. Cross-checking with YearsOnList shows the blanks mean two different things: 13 companies are new to the list and 42 kept their rank.

The ratios point you to the columns that need attention; interpreting them needs business knowledge.


:::{figure} images/employee_histogram.png
:label: fig-employee-histogram
:alt: A histogram of employee counts on a log scale, peaking near 100,000, with annotations for Rajesh Exports at 141 employees and Walmart at 2.1 million.
Plots reveal special observations: the distribution of employee counts on a log scale. Both extremes are real companies.
:::

## Profiling tools

Profiling tools automate the mechanical work: they generate statistical summaries and frequency distributions of values and formats, and for text they can count words or score sentiment. Without them, analysts write their own code, documentation and summaries for every dataset.

- **Commercial tools**, such as those from Informatica, IBM and Oracle, are usually bundled with enterprise data platforms.
- **Community editions** with limited features exist for non-commercial use, such as Talend Open Studio, the Pentaho community edition and the Ataccama ONE profiler.
- **Programming libraries** in Python and R produce a full profile in a few lines.

### Python: fg-data-profiling

**fg-data-profiling** (a community-maintained continuation of the library first released as pandas-profiling and later ydata-profiling) generates an interactive report of every column, plus correlations, interactions between pairs of columns, missing-value patterns and duplicate rows. In Google Colab or Jupyter:

```python
!pip install fg-data-profiling        # then restart the runtime if asked

import pandas as pd
from data_profiling import ProfileReport

df = pd.read_csv("https://raw.githubusercontent.com/KarAnalytics/datasets/master/FortuneGlobal500_2024_cleaned.csv")
profile = ProfileReport(df, title="Fortune Global 500, 2024")
profile.to_notebook_iframe()                     # view the report in the notebook
profile.to_file("fortune500_profiling.html")     # or save it as a web page
```

The report flags, for example, the missing values in RankCng and ProfitsPerCng, the skewed distributions of revenue and employees, and a strong non-linear relationship between rank and revenue (rank is defined by revenue, so this is expected and confirms the data is consistent).

### R: DataExplorer

The **DataExplorer** package produces a similar report with one command:

```r
library(DataExplorer)

f500 <- read.csv("https://raw.githubusercontent.com/KarAnalytics/datasets/master/FortuneGlobal500_2024_cleaned.csv")

create_report(f500)                    # a full profile as an HTML report
create_report(f500, y = "Rank")        # relate every variable to an outcome

plot_boxplot(f500, by = "HqCountry")   # distributions of numeric columns by category
plot_missing(f500)                     # missing values by column
```

Other packages worth exploring: **Sweetviz** and **D-Tale** in Python; **skimr** and **visdat** in R; and standalone tools such as **Data Wrangler** (a VS Code extension) and **KNIME**.

### SQL: profiling in the database

When the data is already in a database, SQL aggregate functions are the most portable profiler. Remember that `COUNT(column)` counts only non-NULL values. Profiling the `SH.CUSTOMERS` table (see [The SH Sample Schema](../appendix/sh-schema.md)) in Oracle:

```sql
SELECT COUNT(*)                          AS total_rows,
       COUNT(cust_income_level)          AS income_actual,
       COUNT(DISTINCT cust_income_level) AS income_cardinality,
       MIN(cust_year_of_birth)           AS oldest_birth_year,
       MAX(cust_year_of_birth)           AS youngest_birth_year,
       COUNT(DISTINCT cust_credit_limit) AS credit_limit_values,
       COUNT(DISTINCT cust_email)        AS distinct_emails
FROM   sh.customers;
```

In the classic SH data, this returns 55,500 customers; 55,459 have an income level (41 are NULL, so completeness is 99.9%), in 12 bands; birth years run from 1913 to 1990; there are only 8 distinct credit limits; and only 1,699 distinct email addresses for 55,500 customers. The last two findings are profiling discoveries: credit limits are set in fixed tiers, and email addresses are synthetic and shared by many customers, so they cannot identify a customer in this sample data.

## A short primer on visualization

Plots are part of every profile:

- **Histograms** show the distribution of one variable.
- **Bar charts** compare groups.
- **Line charts** show trends over time.
- **Scatter plots** show the relationship between two (or three) variables.
- **Heatmaps** use color for intensity, such as a correlation matrix.
- **Pie charts** show proportions of a whole; use them sparingly, for few categories.

Popular tools include Excel and Google Sheets, Tableau, Power BI, D3.js, and Python and R.

## Common mistakes

- **Profiling once.** Profile again after every processing step; fixes introduce new problems.
- **Treating the profile as the answer.** "11% missing" is a finding; why it is missing, and what to do about it, is the analysis.
- **Ignoring disguised missing values.** "N/A", 0 and 9999 hide in columns that look 100% complete.
- **Flagging every outlier as an error.** Rajesh Exports really does have 141 employees; Walmart really does have 2.1 million.
- **Trusting documentation over data.** Profiling exists because metadata and reality drift apart.

:::{warning} Check the AI
GenAI assistants can summarize a profiling report well, and will often add recommendations such as "impute the 11% missing RankCng values with the median" or "drop EmployeeCount outliers beyond three standard deviations". Treat these as hypotheses, not actions. The assistant sees the numbers, not the business: it does not know that a blank rank change means "unchanged" for most companies, or that the smallest employer on the list is a real company. For each recommendation, ask which rows it affects and look at some of them.
:::

## Practice

**1. Ratios.** A `CUSTOMER_PHONE` column has 10,000 rows: 600 NULL, 400 containing "000-000-0000", and 8,700 distinct real numbers. Compute missing, actual, completeness, uniqueness and distinctness. What do they suggest?

:::{note} Answer
:class: dropdown
Missing = 400 (the placeholder), actual = 10,000 - 600 - 400 = 9,000. Completeness = 9,000 / 10,000 = 90%. Uniqueness = 8,700 / 10,000 = 87%. Distinctness = 8,700 / 9,000 = 96.7%. Some real numbers are shared (households or businesses with several customers), and the placeholder makes the column look 94% complete (only NULLs counted) when it is really 90% complete.
:::

**2. Profile in Python or R.** Run the fg-data-profiling (or DataExplorer) code above on the Fortune file. List three warnings or observations from the report, and for each, say whether it is a data problem, a feature of the business, or both.

**3. Profile in SQL.** In your Oracle Cloud account, write one query that profiles `SH.PRODUCTS`: the number of products, the number of distinct categories and subcategories, the minimum and maximum list price, and how many products have a NULL minimum price.

:::{note} One possible answer
:class: dropdown
```sql
SELECT COUNT(*)                         AS products,
       COUNT(DISTINCT prod_category)    AS categories,
       COUNT(DISTINCT prod_subcategory) AS subcategories,
       MIN(prod_list_price)             AS min_list_price,
       MAX(prod_list_price)             AS max_list_price,
       COUNT(*) - COUNT(prod_min_price) AS null_min_price
FROM   sh.products;
```
:::

**4. Plan a profile.** For the Kansas City 311 calls (from the [data processing chapter](../data-processing/intro.md)), write a profiling plan: three table-level checks, five column-level checks (name the columns), and two business rules to test (for example, a call cannot be closed before it was opened).

## Key terms

information extraction, data profiling, data quality, fitness for use, GIGO, metadata, master data management, NULL, missing value, actual value, cardinality, completeness, uniqueness, distinctness, domain integrity, structural integrity, referential integrity, outlier, special observation, fg-data-profiling (pandas-profiling, ydata-profiling), DataExplorer

## Further reading

- fg-data-profiling, [repository and documentation](https://github.com/Data-Centric-AI-Community/fg-data-profiling).
- DataExplorer, [introduction vignette](https://cran.r-project.org/web/packages/DataExplorer/vignettes/dataexplorer-intro.html).
