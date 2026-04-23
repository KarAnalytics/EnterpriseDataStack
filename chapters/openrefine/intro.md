# Data Processing with OpenRefine

OpenRefine (née Google Refine) is a free, open-source desktop tool for exploring, cleaning, and reshaping messy tabular data. It runs locally in your browser and shines at the kind of interactive, iterative cleanup that is painful in SQL and noisy in pandas.

## When to reach for OpenRefine

- You have a flat file (CSV, Excel, TSV, JSON) that needs cleanup before it's worth loading into a database
- You're looking at free-text fields (city names, company names, job titles) with lots of spelling variants
- You want a reversible, auditable cleanup history — every operation can be undone and exported as a script
- You want a GUI while still being able to write expressions in a real language (GREL, Python, Jython, Clojure)

## Core workflow

1. **Create a project** — drop in a file; OpenRefine parses it and shows a preview
2. **Facet** — click a column, choose "Text facet" or "Numeric facet" to see distinct values and their counts
3. **Cluster** — use OpenRefine's algorithms (key collision, nearest neighbor) to group near-duplicate strings and merge them
4. **Transform** — apply GREL expressions to clean values, split columns, extract substrings, parse dates
5. **Export** — back to CSV / Excel / TSV / JSON, or push directly to a Google Sheet

## Faceting

Facets are dynamic filters over a column. Text facets show every distinct value; numeric facets show a histogram; timeline facets show a date distribution. Facets are the single fastest way to spot inconsistencies — a "Gender" column with values `M`, `F`, `Male`, `female`, `m `, `f ` (with trailing space) reveals itself instantly.

## Clustering — the killer feature

Clustering groups values that *look similar but are written differently*. OpenRefine offers multiple methods:

- **Key collision** — fingerprint, n-gram fingerprint, phonetic (Metaphone3, Cologne, Beider-Morse, Daitch-Mokotoff)
- **Nearest neighbor** — Levenshtein, PPM

Example: a "Company" column containing `IBM`, `I.B.M.`, `International Business Machines`, `Ibm Corp.`, and `ibm inc` will cluster together. You pick the canonical name and merge the cluster with one click.

## GREL — General Refine Expression Language

A small expression language for transforms. A few common patterns:

```
value.trim().toLowercase()                           // clean strings
value.replace(/\s+/, ' ')                            // collapse whitespace (regex)
value.toDate('MM/dd/yyyy').toString('yyyy-MM-dd')    // normalize date format
if(isBlank(value), cells['OtherCol'].value, value)   // conditional fill
value.split('@')[1]                                  // extract domain from email
cells['First'].value + ' ' + cells['Last'].value     // combine columns
```

## Provenance and reproducibility

Every action lives in OpenRefine's undo/redo history. You can export that history as a JSON **operation recipe** and re-apply it to the next batch of data — the same cleanup, reproducibly, on new inputs. This is what makes OpenRefine defensible for enterprise use, not just one-off analysis.

## Limits

- In-memory processing — starts to struggle above a few million rows
- Single user — no collaborative editing
- Schema-less — good for cleanup, not for long-term storage

The typical pattern is: use OpenRefine to clean, export to CSV, load to a database.

## Learning outcomes

- Install OpenRefine and create a project from a CSV or Excel file
- Use text, numeric, and timeline facets to diagnose data quality issues
- Cluster and merge near-duplicate values
- Write GREL expressions for common transforms
- Export a cleanup recipe for reproducible reuse

