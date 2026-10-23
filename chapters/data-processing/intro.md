# Data Processing

Real data is never clean. Files arrive with inconsistent spelling, dates in several formats, blanks that mean different things, and characters that turn into question marks. **Data processing** is the work of adjusting data until it is organized and ready to produce information. This chapter covers the kinds of data you will meet, the three classic data cleaning tasks, and a checklist of common processing steps, with a worked example in Python.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Define data processing, and recognize that the term means different things to different audiences.
2. Identify the types of data and the common data types of columns.
3. Apply the three data cleaning tasks: parsing, correction and standardization.
4. Plan the processing steps a dataset needs, and carry out basic steps in Python.
:::

:::{note} Think about it
Kansas City's 311 Call Center is a single point of contact for residents to report problems, request city services and ask questions. Suppose your team is asked to analyze a year of 311 calls and identify systematic patterns. Download the data from the city's open data portal ([data.kcmo.org](https://data.kcmo.org)), look at the rows and columns, and list the data challenges you expect *before* any analysis: missing values, inconsistent categories, addresses, dates and times, duplicates. Keep the list; you will meet most of these problems in this chapter and the next two.
:::

## What is data processing?

In analytics and technology teams, **data processing** is the process of adjusting data to make it organized, easier to read, and ready for downstream processing that generates information and knowledge: the set of operations, such as collection, organization, storage and transformation, that convert data into useful information.

Outside analytics circles the term is often broader: *any* operation performed on data, across its whole life from collection to analysis or storage. When a client asks for "data processing", clarify which meaning they intend.

Processing takes time, often most of the time in an analytics project, and every application needs a different set of processing steps. It goes hand in hand with [data profiling](../data-profiling/intro.md): profiling finds the problems, processing fixes them, and profiling again confirms the fix.

## Types of data

| Type | Description | Example |
|---|---|---|
| **Numeric** | numbers | employee salaries |
| **String** | short sequences of letters, digits or symbols | a product code, a gender code |
| **Text** | strings arranged in a logical order, such as sentences | product reviews, call-center notes |
| **Multimedia** | images (matrices of pixels with brightness and color values), audio (time series of numbers) and video (images in temporal order) | photos of damaged packages |
| **Spatial** | two- or three-dimensional data with geographic coordinates or contours | store locations, delivery routes |

### Data types of columns

Every column in a table or data frame has a **data type**, which determines what values it can hold and what you can do with them. The names differ by system, but the ideas are the same:

| Kind of value | Oracle SQL | Python (pandas) | R |
|---|---|---|---|
| Whole numbers | `NUMBER(p)`, `INTEGER` | `int64` | `integer` |
| Exact decimals (money) | `NUMBER(p,s)` | `float64` (or `Decimal`) | `numeric` |
| Approximate decimals | `BINARY_DOUBLE`, `FLOAT` | `float64` | `numeric` (double) |
| True/false | `BOOLEAN` (recent versions) or a `CHAR(1)` flag | `bool` | `logical` |
| Fixed or variable text | `CHAR(n)`, `VARCHAR2(n)` | `object` or `string` | `character` |
| Categories | text with a check constraint or lookup table | `category` | `factor` |
| Dates and times | `DATE`, `TIMESTAMP` | `datetime64` | `Date`, `POSIXct` |

A column stored with the wrong type causes quiet errors: numbers stored as text sort as text (`"100"` comes before `"9"`), and dates stored as text sort by month first. Checking and correcting types is one of the first processing steps.

## Three data cleaning tasks

Data cleaning is usually described as three tasks, applied in order. Consider one messy line from a conference sign-up sheet:

> `Dr. Maria  L. Okafor, VP Ops; Prairie Wind Energy Inc; 1200 Main St, Ste 400, Lawrence KS`


:::{figure} images/cleaning_steps.png
:label: fig-cleaning-steps
:alt: A flow of five boxes: raw text, then parsing, correction and standardization, ending in a clean record.
The three data cleaning tasks, applied in order. Source: {download}`cleaning_steps.drawio <images/cleaning_steps.drawio>`.
:::

**1. Parsing** locates and separates the individual data elements in text. It has been studied in computer science for decades; simple cases use [regular expressions](../regular-expressions/intro.md), and context-sensitive cases use natural language processing.

| Element | Parsed value |
|---|---|
| Title | Dr. |
| First name | Maria |
| Middle initial | L. |
| Last name | Okafor |
| Job title | VP Ops |
| Company | Prairie Wind Energy Inc |
| Street | 1200 Main St, Ste 400 |
| City | Lawrence |
| State | KS |

**2. Correction** fixes values that are wrong or missing:

- **Missing values**: use a default for values that do not apply, a typical value (mean, median or mode), or a more complex prediction from other fields. Here the postal code is missing and can be looked up from the address: 66044.
- **Conflicting values**: when two sources disagree, prefer the more recent value or the more credible source.
- **Data types and timestamps**: fix values stored with the wrong type, and account for local time zones.

**3. Standardization** converts values into preferred formats, using both standard and custom business rules:

| Element | Corrected | Standardized |
|---|---|---|
| Job title | VP Ops | Vice President, Operations |
| Company | Prairie Wind Energy Inc | Prairie Wind Energy, Inc. |
| Street | 1200 Main St, Ste 400 | 1200 Main Street, Suite 400 |
| State | KS | KS |
| Postal code | 66044 | 66044 |
| Country | (none) | USA |

Standardization is what lets you count, join and group reliably: "VP Ops", "V.P. Operations" and "Vice President of Operations" become one value.

## Common data processing tasks

A checklist of steps to consider for any dataset:

- **Scope and enrich**: exclude irrelevant variables, and enhance the dataset with external sources (for example, add weather data to sales).
- **Deduplicate** records.
- **Verify and correct** data types and units of measure.
- **Standardize** locations (addresses, city and country names) and timestamps; create a date variable from a timestamp, and generate variables such as day of week or holiday.
- **Handle missing values.**
- **Categorical variables**: group too many levels into a feasible number, and create one-hot (dummy) variables for modeling.
- **Numerical variables**: treat outliers (exclude, cap or replace), discretize into categories using a suitable logic, or transform (for example, to a standard normal or logistic scale).
- **Data augmentation**: generate artificial records from existing data, for example to balance rare cases.

:::{note} A note on Excel
Excel is a valuable first tool for profiling and processing: open the data, look at it, and use filters, formulas, Text to Columns and the Analysis ToolPak. For larger or repeatable work, use open-source tools such as [OpenRefine](../openrefine/intro.md), Python or R. If a file is too big for Excel, open a sample of it in a CSV or TSV format first. Looking at the data before using any other tool is always worth the time.
:::

## Worked example: processing the Fortune Global 500

The Fortune Global 500 list for 2024 ranks the world's largest companies by revenue. A copy is at `https://raw.githubusercontent.com/KarAnalytics/datasets/master/FortuneGlobal500_2024.csv`, with 13 columns: rank, company, revenue, profits, assets and employees, the change in rank and in revenue and profits since the previous year, years on the list, and headquarters city, country and industry.

**Step 1: read the file with the right encoding.** Reading it the default way fails:

```python
import pandas as pd
url = "https://raw.githubusercontent.com/KarAnalytics/datasets/master/FortuneGlobal500_2024.csv"
df = pd.read_csv(url)
# UnicodeDecodeError: 'utf-8' codec can't decode byte 0xfc in position 8142
```

Byte `0xfc` is "ü" in the Windows-1252 encoding (in *Düsseldorf*). The file was saved by a Windows program, so read it with that encoding:

```python
df = pd.read_csv(url, encoding="cp1252")
df.shape          # (500, 13)
```

**Step 2: look for damage the encoding already caused.** Windows-1252 cannot represent every character. Searching for question marks finds one casualty: the Polish oil company Orlen is headquartered in "P?ock", which should be Płock. The original character was lost when the file was saved, so it has to be corrected by hand:

```python
df.loc[df["Hqcity"].str.contains("?", regex=False), ["Company", "Hqcity"]]   # Orlen, P?ock
df.loc[df["Hqcity"] == "P?ock", "Hqcity"] = "Płock"
```

**Step 3: decide what each blank means.** `RankCng` (the change in rank since last year) is blank for 55 companies, and never zero. Comparing with `YearsOnList` shows that the blank means two different things: 13 companies are new to the list (a change cannot be computed), and the other 42, including Walmart at rank 1, kept the same rank (the change is zero). Encode the difference explicitly instead of letting an analyst guess:

```python
df["NewToList"] = df["YearsOnList"] == 1
df.loc[df["RankCng"].isna() & ~df["NewToList"], "RankCng"] = 0
df["RankCng"].isna().sum()     # 13: only the new entrants remain blank
```

**Step 4: derive variables.** Revenue per employee is a useful efficiency measure (revenue is in millions of dollars):

```python
df["RevenuePerEmployee"] = df["RevenueMil"] * 1_000_000 / df["EmployeeCount"]
```

**Step 5: discretize and encode.** Group companies into size bands by employee count, then create one-hot columns for modeling:

```python
df["SizeBand"] = pd.cut(df["EmployeeCount"], bins=[0, 50_000, 200_000, float("inf")],
                        labels=["Under 50k", "50k to 200k", "Over 200k"])
df["SizeBand"].value_counts()   # Under 50k: 137, 50k to 200k: 258, Over 200k: 105
df = pd.get_dummies(df, columns=["SizeBand"])
```

**Step 6: save in a standard encoding.**

```python
df.to_csv("fortune_processed.csv", index=False, encoding="utf-8")
```

None of these steps is complicated. The skill is in noticing that they are needed: a file that loads, a blank that looks like "missing", and a "?" in a city name are all easy to miss.

## Common mistakes

- **Reading with `errors="ignore"` or the wrong encoding** to make an error go away. Characters disappear silently.
- **Treating every blank as "unknown".** A blank can mean zero, not applicable, not yet known or not collected, and each needs different handling.
- **Filling missing values with the mean by default.** Ask first why the values are missing.
- **Standardizing without a rule book.** Write the mapping down (for example, every spelling of each state maps to its two-letter code) so that the next file is processed the same way.
- **Processing the only copy.** Keep the raw file untouched and save processed data separately.

:::{warning} Check the AI
Ask a GenAI assistant to "clean the Fortune file" and it will usually produce code that runs: it may add `encoding_errors="ignore"`, fill `RankCng` with the median, and drop rows with missing values. Each choice is defensible in general and wrong here: the first deletes letters from city names, the second invents rank changes for new companies, and the third throws away 85 of the world's largest companies. Before you accept processing code, ask what each step does to the specific values that will be affected, and check a few of those rows yourself.
:::

## Practice

**1. Clean one record.** Parse, correct and standardize: `Mr JOHN q. public jr., sr mgr acctg, Acme Corp., 55 W 3rd ave apt 2b, kansas city, mo 64105`.

:::{note} One possible answer
:class: dropdown
Parsed: title Mr; first name John; middle initial Q.; last name Public; suffix Jr.; job title sr mgr acctg; company Acme Corp.; street 55 W 3rd ave apt 2b; city kansas city; state mo; postal code 64105. Standardized: job title Senior Manager, Accounting; company Acme Corporation; street 55 West 3rd Avenue, Apartment 2B; city Kansas City; state MO; country USA. The name is converted from all capitals to proper case.
:::

**2. Choose a type.** Which data type would you use for: (a) a US zip code; (b) a product price; (c) whether an order was gifted; (d) the time a 311 call was received?

:::{note} Answer
:class: dropdown
(a) Text (`VARCHAR2(10)`), not a number: zip codes can start with 0 and can include a hyphen and four more digits, and you never do arithmetic on them. (b) An exact decimal such as `NUMBER(10,2)`. (c) Boolean (or a `Y`/`N` flag). (d) `TIMESTAMP`, with the time zone recorded or standardized.
:::

**3. Blanks.** In the Fortune file, `ProfitsPerCng` (percentage change in profits) is blank for 37 companies, including Amazon, Ford, BP and Berkshire Hathaway, all of which lost money in the previous year. Why might the percentage change be blank, and how would you process the column?

:::{note} Answer
:class: dropdown
A percentage change from a negative (or zero) base is not meaningful, so the source leaves it blank. Do not fill it with a number. Keep the blank, add a flag column explaining it (for example, `PrevYearLoss`), and, if a comparison is needed, compute the change in dollars instead, which is defined for every company that was on the list both years.
:::

**4. Plan the 311 data.** Using the data challenges you listed for the Kansas City 311 calls, write a processing plan of five to eight steps in the order you would do them, naming the columns each step affects.

## Key terms

data processing, numeric, string, text, multimedia and spatial data, data type, parsing, correction, standardization, missing value, conflicting value, deduplication, discretization, one-hot encoding, outlier, data augmentation, character encoding (UTF-8, Windows-1252)

## Further reading

- pandas documentation, [Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html).
