# Data Processing with OpenRefine

Writing code is not the only way to clean data. **OpenRefine** is a free desktop tool that lets you explore and fix messy data interactively, with features (facets and clustering) that make inconsistencies jump out, and a complete record of every change you make. This chapter introduces OpenRefine's core workflow and walks through cleaning the Fortune Global 500 file from the [previous chapter](../data-processing/intro.md).

:::{tip} Learning objectives
After this chapter you should be able to:

1. Create an OpenRefine project from a file, choosing the right import options.
2. Explore data with text, numeric and blank facets and text filters.
3. Find and merge variant spellings with clustering.
4. Transform cells and add columns with common transforms and GREL expressions.
5. Save a project's operation history and export the cleaned data.
:::

## What OpenRefine is

OpenRefine (formerly Google Refine) is a standalone, open-source desktop application for data cleanup and transformation. It runs on your own computer and opens in your web browser, so your data never leaves your machine. Download it from [openrefine.org](https://openrefine.org), start it, and it opens a page at `http://127.0.0.1:3333`.

It shines at three things:

- **Facets**: instant summaries of a column's values, which double as filters.
- **Clustering**: finding values that are probably the same thing typed differently.
- **Repeatable transformations**: every change is recorded as an operation that can be undone, replayed, or applied to the next file.

OpenRefine is one of many processing tools. Almost everything it does can also be programmed in Python or R, which is better for large or automated jobs. OpenRefine is best for exploring a new dataset and fixing it interactively, especially for people who do not program.

## The workflow


:::{figure} images/openrefine_workflow.png
:label: fig-openrefine-workflow
:alt: Five steps in a row: import, explore, cluster, transform and export, above a bar for the undo and redo history that records every operation.
The OpenRefine workflow. Every operation is recorded in the project history. Source: {download}`openrefine_workflow.drawio <images/openrefine_workflow.drawio>`.
:::

### 1. Create a project

1. Click **Create Project**, then choose **This Computer** (upload a file) or **Web Addresses (URLs)**.
2. On the preview screen, check the **parsing options**: the file type (CSV, Excel, JSON, ...), the **character encoding**, whether the first row holds column headers, and whether to **parse cell text into numbers and dates**. The preview updates as you change them.
3. Name the project and click **Create Project »** at the top right.

### 2. Explore with facets and filters

Every column has a menu (the small triangle in its header). **Facet** opens a summary panel on the left:

- A **text facet** lists every distinct value with its count. Click a value to show only those rows; hover over it and choose **edit** to change every cell with that value at once.
- A **numeric facet** shows a histogram; drag the handles to select a range, and see how many rows are non-numeric, blank or errors.
- **Customized facets** include **facet by blank** (which rows are empty?) and **timeline facets** for dates.
- A **text filter** shows only the rows whose cells contain a string.

Facets combine: select a value in one facet and a range in another, and OpenRefine shows the rows that satisfy both. Any edit applies only to the rows currently selected.

### 3. Cluster similar values

**Clustering** groups values that are probably the same thing with different spellings, capitalization, spacing or punctuation. Open it from a text facet's **Cluster** button, or from **Edit cells > Cluster and edit**. Suppose a survey's City column contains:

| Value | Count |
|---|---|
| Kansas City | 412 |
| kansas city | 37 |
| KANSAS CITY | 12 |
| Kansas  City (two spaces) | 5 |
| Kansas City. | 3 |
| Kansas Cty | 2 |
| KC | 9 |

OpenRefine offers two families of methods:

- **Key collision** methods compute a key for each value and group values with the same key. The default, **fingerprint**, trims the value, lowercases it, removes punctuation, and sorts its words, so the first five values above all get the key `city kansas` and form one cluster. Phonetic keys (such as Metaphone) group values that *sound* alike.
- **Nearest neighbor** methods group values within a small edit distance of each other, such as **Levenshtein** distance. This catches the typo "Kansas Cty", which differs from "Kansas City" by one letter.

For each cluster, review the values, tick **Merge?**, type the value to keep, and click **Merge selected**. Clustering suggests; you decide. "KC" will not be found by either method, so it needs a rule of its own (edit it in the text facet).

If a text facet shows no clusters, that is good news: the column is already consistent.

### 4. Transform cells

The **Edit cells > Common transforms** menu handles the everyday fixes: **Trim leading and trailing whitespace**, **Collapse consecutive whitespace**, **To titlecase**, **To uppercase**, **To lowercase**, **To number**, **To date** and **To text**.

For anything else, **Edit cells > Transform...** accepts an expression in **GREL** (General Refine Expression Language), where `value` is the current cell's value:

| Goal | GREL expression |
|---|---|
| Trim spaces | `value.trim()` |
| Title case | `value.toTitlecase()` |
| Replace text | `value.replace("St.", "Street")` |
| Parse a US-style date | `value.toDate("MM/dd/yyyy")` |
| Write a date as `yyyy-MM-dd` | `value.toString("yyyy-MM-dd")` |
| Use another column | `cells["EmployeeCount"].value` |

Dates matter: a column of `MM/DD/YYYY` text, common in exported files, is not a standard timestamp format for most enterprise systems. Convert it to a real date (**To date** or `toDate`), and export it in ISO format (`2026-03-14`).

**Edit column > Add column based on this column...** creates a new column from a GREL expression, and **Split into several columns...** parses one column into many (for example, "City, State").

### 5. Keep the history and export

The **Undo / Redo** tab lists every operation since the project was created. Click any earlier step to go back to it. **Extract...** saves the operations as JSON, and **Apply...** replays them on another project, so next month's file can be cleaned with one click. This record is also documentation: it shows exactly how the cleaned data was produced.

When you are done, **Export** (top right) writes the data as CSV, TSV, Excel or other formats. Exported data contains only the rows matching the current facets, so clear your facets first if you want everything.

## Worked example: cleaning the Fortune Global 500

1. **Import with the right encoding.** Create a project from the URL `https://raw.githubusercontent.com/KarAnalytics/datasets/master/FortuneGlobal500_2024.csv`. In the parsing options, set **Character encoding** to `windows-1252` and tick **Parse cell text into numbers, dates, ...**. Check the preview: *Düsseldorf* and *São Paulo* should look right. If they show odd symbols, the encoding is wrong.

2. **Find the damaged city.** On **Hqcity**, apply a **text filter** for `?`. One row remains: Orlen, in "P?ock". Click the cell, choose **edit**, and type `Płock`. Remove the filter.

3. **Inspect the industry column.** A **text facet** on **IndustrySector** shows 57 values. Sorting by name reveals two vocabularies mixed together: detailed industries ("Commercial Banks", "Petroleum Refining") and a handful of broad sectors ("Financials", "Industrials", "Materials", "Energy", "Technology", "Health Care", "Retail"). Two labels name the same industry: "Life and Health Insurance" (16 companies) and "Insurance: Life, Health (stock)" (2). In the facet, edit the second label to `Life and Health Insurance`; both rows change at once. Mapping the broad sectors onto the detailed list needs a business rule (which detailed industry does each company belong to?), so record it as an open issue rather than guessing.

4. **Try clustering.** Click **Cluster** on the IndustrySector facet. Fingerprint finds no clusters: the problem in this column is two vocabularies, not typos. Clustering finds spelling variants; it cannot find synonyms.

5. **Check the numbers.** A **numeric facet** on **EmployeeCount** shows a long right tail (Walmart, 2.1 million employees) and a tiny minimum: Rajesh Exports, with 141 employees. Before treating it as an error, check: it is a gold refiner and trader with very few staff for its revenue. A profile flags such values; a person decides what they mean.

6. **Mark the blanks.** On **RankCng**, choose **Facet > Customized facets > Facet by blank**: 55 rows are blank. Add a numeric facet on **YearsOnList** alongside it and select 1: 13 of the blanks are companies new to the list. The other 42 kept their rank (see the [previous chapter](../data-processing/intro.md)).

7. **Add a column.** On **RevenueMil**, choose **Edit column > Add column based on this column...**, name it `RevenuePerEmployee`, and enter `value * 1000000 / cells["EmployeeCount"].value`.

8. **Save the recipe and export.** In **Undo / Redo**, click **Extract...** and save the JSON with your project notes. Clear all facets, then **Export > Comma-separated value**.

## Common mistakes

- **Skipping the preview.** A wrong encoding or a missed header row is easiest to fix before the project is created.
- **Editing with a facet still selected.** The edit applies only to the matching rows, which may be what you want, or may not.
- **Merging clusters without reading them.** "Austin" and "Boston" are not the same city, however close the algorithm thinks they are.
- **Exporting a filtered view.** Clear the facets before exporting the full dataset.

:::{warning} Check the AI
GenAI assistants can write GREL and suggest clustering settings, but they often mix GREL up with Python or JavaScript syntax (for example, writing `value.title()` or `value.strip()`, which GREL does not have). Test every expression on the preview in the Transform dialog before applying it; the preview shows the result for the first rows. And never paste a dataset with personal information into an online assistant to have it "cleaned": OpenRefine runs locally precisely so your data stays on your machine.
:::

## Practice

**1. Facets.** In the Fortune project, which headquarters country has the most companies, and how many companies are headquartered in Japan?

:::{note} Answer
:class: dropdown
Use a text facet on **HqCountry** and sort by count: the United States has 140 companies, China 127 and Japan 40.
:::

**2. Clustering.** Which method would group each pair: (a) "Smith & Sons" and "smith and sons"; (b) "Catherine" and "Kathryn"; (c) "Johnson" and "Jonhson"?

:::{note} Answer
:class: dropdown
(a) Neither fingerprint nor nearest neighbor reliably equates "&" with "and"; standardize "&" to "and" first (`value.replace("&", "and")`), then fingerprint groups them. (b) A phonetic key collision method (such as Metaphone). (c) Nearest neighbor with Levenshtein distance. A transposition counts as a distance of 2, so raise the radius from its default of 1 to 2.
:::

**3. GREL.** Write GREL expressions to (a) turn `"  mid-west REGION "` into `Mid-West Region`; (b) turn the text `03/14/2026` into the text `2026-03-14`.

:::{note} Answer
:class: dropdown
(a) `value.trim().toTitlecase()`. (b) `value.toDate("MM/dd/yyyy").toString("yyyy-MM-dd")`.
:::

**4. A repeatable recipe.** Clean the Fortune file as in the worked example, extract the operation history as JSON, then create a new project from the same file and **Apply...** the JSON. Do you get the same result? Which step did not replay as you expected, and why?

## Key terms

OpenRefine, project, parsing options, character encoding, facet (text, numeric, blank, timeline), text filter, clustering, key collision, fingerprint, phonetic key, nearest neighbor, Levenshtein distance, common transforms, GREL, operation history, export

## Further reading

- OpenRefine, [User Manual](https://openrefine.org/docs), including the pages on facets, clustering and GREL.
