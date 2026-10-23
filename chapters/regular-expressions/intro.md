# Regular Expressions

How would you find every phone number in ten thousand customer comments, check that every product code has two letters and four digits, or pull the domain out of a column of email addresses? **Regular expressions** (regex) describe patterns of characters, and they are one of the most expressive tools for extracting and validating data. This chapter covers the basics: enough to read and write the patterns you will need for data wrangling, in Python and in Oracle SQL.

:::{tip} Learning objectives
After this chapter you should be able to:

1. Explain the parts of a regular expression: literals, metacharacters and escape sequences.
2. Read and write patterns using quantifiers, anchors, character classes and alternation.
3. Use regular expressions in Python with the `re` library and pandas.
4. Use regular expressions in Oracle SQL with `REGEXP_LIKE`, `REGEXP_SUBSTR` and `REGEXP_REPLACE`.
:::

## What a regular expression is

A **regular expression** is a pattern used to match combinations of characters in strings. Regular expressions are available in almost every programming language and database. They can take a long time to master, but the basics, covered here, already solve most data wrangling problems.

A pattern (the **search expression**) is applied to a **target string**, and it is built from three kinds of pieces:

- **Literals**: ordinary characters that match themselves, such as `c`, `o`, `m`.
- **Metacharacters**: characters with special meaning, such as `^ $ . [ ] + * ? { } | ( )`.
- **Escape sequences**: a backslash that turns a metacharacter back into a literal (`\.` matches an actual dot) or introduces a shorthand class (`\d` for a digit).

For example, `^[a-z]+\.com$` matches `abc.com`: the start of the string (`^`), one or more lowercase letters (`[a-z]+`), a literal dot (`\.`), the letters `com`, and the end of the string (`$`).


:::{figure} images/regex_anatomy.png
:label: fig-regex-anatomy
:alt: The pattern for abc.com split into five colored pieces with labels: start anchor, character set with quantifier, escaped dot, literal com, end anchor.
The anatomy of a regular expression. Source: {download}`regex_anatomy.drawio <images/regex_anatomy.drawio>`.
:::

## Metacharacters

| Metacharacter | Type | Meaning |
|---|---|---|
| `?` | quantifier | the preceding item 0 or 1 time |
| `*` | quantifier | the preceding item 0 or more times |
| `+` | quantifier | the preceding item 1 or more times |
| `{n}` | quantifier | the preceding item exactly n times |
| `{n,m}` | quantifier | the preceding item at least n and at most m times |
| `[ ]` | character set | any one of the enclosed characters; a hyphen gives a range, as in `[0-9]` |
| `[^ ]` | negated set | any one character *not* enclosed |
| `^` | anchor | the start of the string (when it is the first character of the pattern) |
| `$` | anchor | the end of the string (when it is the last character of the pattern) |
| `.` | wildcard | any single character except a newline |
| `\|` | alternation | the pattern on the left *or* the pattern on the right |
| `( )` | group | treats several characters as one item, and captures what they matched |
| `\` | escape | makes the next metacharacter literal, or starts a shorthand class |

Shorthand classes save typing:

| Class | Matches | Equivalent |
|---|---|---|
| `\d` | a digit | `[0-9]` |
| `\w` | a word character | `[A-Za-z0-9_]` |
| `\s` | a whitespace character | space, tab, newline |
| `\b` | a word boundary (the edge of a word; matches no character) | |
| `\D`, `\W`, `\S` | the opposite of `\d`, `\w`, `\s` | |

### Examples

| Pattern | Target strings | Result |
|---|---|---|
| `colou?r` | color, colour | both match: `u` is optional |
| `\d{3}-\d{4}` | 864-0123, 8640-123 | only the first |
| `^[A-Z]{2}\d{4}$` | KS1234, ks1234, KS12345 | only the first: two capital letters then exactly four digits, nothing more |
| `^(19\|20)\d{2}$` | 1999, 2026, 2160 | the first two: a four-digit year starting with 19 or 20 |
| `^\d{5}(-\d{4})?$` | 66045, 66045-7585, 6604 | the first two: a US zip code with an optional four-digit extension |
| `^win` | erwin, window | only *window* (starts with "win") |
| `win$` | erwin, window | only *erwin* (ends with "win") |
| `[^0-9]+` | 123, abc, a456 | *abc* and *a456* contain at least one non-digit |
| `dog\|cat\|frog` | a dog, cat friend, frogman | all three |

A few longer patterns show how the pieces combine:

| Field | Pattern |
|---|---|
| User name (3 to 16 letters, digits, `_` or `-`) | `^[a-z0-9_-]{3,16}$` |
| Hex color (`#a3c113` or `#fff`) | `^#?([a-f0-9]{6}\|[a-f0-9]{3})$` |
| Email address (simple check) | `^[a-z0-9_.-]+@[a-z0-9.-]+\.[a-z]{2,6}$` |

You do not have to memorize everything. Remember the basic symbols, and look up specific patterns when you need them. Regex dialects also differ slightly between Python, R, Oracle SQL and other software, so test a pattern in the tool where you will use it.

## Regular expressions in Python

Python's built-in `re` library provides the main operations:

| Function | Returns |
|---|---|
| `re.search(p, s)` | the first match anywhere in `s` (or `None`) |
| `re.match(p, s)` | a match only at the start of `s` |
| `re.findall(p, s)` | a list of all matches |
| `re.sub(p, r, s)` | `s` with every match replaced by `r` |

Write patterns as raw strings, `r"..."`, so Python does not interpret the backslashes itself.

```python
import re

text = "Contact Jordan at jordan.lee@example.com or 785-555-0142; backup: j_lee@mail.example.org, (785) 555-0199."

re.findall(r"[\w.-]+@[\w.-]+\.[A-Za-z]{2,}", text)
# ['jordan.lee@example.com', 'j_lee@mail.example.org']

re.findall(r"\(?\d{3}\)?[ -]?\d{3}-\d{4}", text)
# ['785-555-0142', '(785) 555-0199']

re.sub(r"\D", "", "(785) 555-0199")      # keep only the digits
# '7855550199'
```

pandas applies the same patterns to whole columns: `str.contains` (true/false), `str.extract` (captured groups into new columns), `str.findall` and `str.replace`.

## Worked example: app store reviews

The file `reviews_ikan.csv` holds 77 public Google Play reviews of the State of Kansas's iKan app, collected in August 2020. The `content` column holds the review text and `at` holds the review time.

```python
import re
import pandas as pd

reviews = pd.read_csv("https://raw.githubusercontent.com/KarAnalytics/datasets/master/reviews_ikan.csv")
text = reviews["content"]
```

**Which reviews are about renewals?** Users write *renew*, *renewal*, *renewals*, *renewed* and *renewing*. One pattern finds them all, ignoring case:

```python
text.str.contains(r"\brenew\w*", flags=re.IGNORECASE).sum()      # 27 of 77 reviews
```

`\b` makes sure the match starts at the beginning of a word, and `\w*` allows any ending.

**Which words are shouted?** Words of three or more capital letters often signal frustration (or an acronym such as DMV):

```python
sorted({w for t in text for w in re.findall(r"\b[A-Z]{3,}\b", t)})
# ['DESCRIMINATING', 'DMV', 'IMO', 'NOT', 'PIN', 'PLATE', 'REAL', 'REG', 'SSL', 'UNINSTALL']
```

Note that regex finds the misspelling *DESCRIMINATING* just as happily as any other word: it matches patterns, not meaning.

**Fix the dates.** The `at` column is text in month/day/year form, such as `8/3/20 16:55`. Sorted as text, the "earliest" review is `1/21/20` and the "latest" is `9/6/19`, both wrong. Capture the parts with groups and rebuild an ISO date:

```python
parts = reviews["at"].str.extract(r"^(\d{1,2})/(\d{1,2})/(\d{2}) (\d{1,2}):(\d{2})$")
reviews["date"] = "20" + parts[2] + "-" + parts[0].str.zfill(2) + "-" + parts[1].str.zfill(2)
reviews["date"].min(), reviews["date"].max()      # ('2018-04-06', '2020-08-03')
```

Every row matched the pattern (no NaN in `parts`), which is itself a useful profiling result: the format is consistent. In practice `pd.to_datetime(reviews["at"], format="%m/%d/%y %H:%M")` does the conversion directly; the regex version shows what such functions do, and works in tools that lack them.

## Regular expressions in Oracle SQL

Oracle provides regex functions that work inside queries:

| Function | Use |
|---|---|
| `REGEXP_LIKE(col, pattern [, 'i'])` | a condition in `WHERE`: does the value match? (`'i'` ignores case) |
| `REGEXP_SUBSTR(col, pattern, position, occurrence, flags, group)` | extract the matching text, or one captured group |
| `REGEXP_REPLACE(col, pattern, replacement)` | replace every match |
| `REGEXP_COUNT(col, pattern)` | count the matches |

For example, which email domains do the customers in the SH sample schema use?

```sql
SELECT REGEXP_SUBSTR(cust_email, '@(.+)$', 1, 1, NULL, 1) AS email_domain,
       COUNT(*)                                          AS customers
FROM   sh.customers
GROUP  BY REGEXP_SUBSTR(cust_email, '@(.+)$', 1, 1, NULL, 1)
ORDER  BY customers DESC;
```

In the classic SH data the answer is two synthetic domains: `company.example.com` (54,500 customers) and `company2.example.com` (1,000). And a validation check, listing values that do *not* look like an email address:

```sql
SELECT cust_id, cust_email
FROM   sh.customers
WHERE  NOT REGEXP_LIKE(cust_email, '^[A-Za-z0-9._-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}$');
```

It returns no rows in SH; on real customer data, it rarely does.

## Common mistakes

- **Forgetting to escape the dot.** `example.com` also matches `exampleXcom`; write `example\.com`.
- **Forgetting the anchors.** `\d{5}` finds five digits *inside* `123456`; `^\d{5}$` requires exactly five.
- **Greedy matches.** `<.+>` on `<b>bold</b>` matches the whole string, not just `<b>`; `<.+?>` (lazy) or `<[^>]+>` stops at the first `>`.
- **Testing only the examples you expect.** Try the edge cases: empty strings, extra spaces, capital letters, accented characters.
- **Trying to validate everything with one regex.** A pattern can check that an email *looks* valid, not that it exists.

:::{warning} Check the AI
GenAI assistants write regular expressions quickly, and a regex that looks plausible is easy to accept without testing. Common problems: patterns that are too loose (an "email" pattern that accepts `a@b`), too strict (a phone pattern that rejects `(785) 555-0199`), written in the wrong dialect (Python's `(?i)` or lookbehinds in a tool that does not support them), or unanchored when they should be anchored. Always ask the assistant for test strings that should *and should not* match, run the pattern on them, and run it on a sample of your real data.
:::

## Practice

**1. Read the pattern.** Which strings does `^[A-Z]{3}-\d{2,3}[a-z]?$` match: `KSU-101`, `KU-101`, `MIZ-12b`, `NEB-1234`, `osu-99`?

:::{note} Answer
:class: dropdown
`KSU-101` and `MIZ-12b`. `KU-101` has only two capital letters; `NEB-1234` has four digits; `osu-99` is lowercase.
:::

**2. Write the pattern.** Write patterns for: (a) a time in 24-hour format such as `09:30` or `23:59`; (b) a US Social Security number written `123-45-6789`; (c) any word ending in *ing*.

:::{note} One possible answer
:class: dropdown
(a) `^([01]\d|2[0-3]):[0-5]\d$`. (b) `^\d{3}-\d{2}-\d{4}$`. (c) `\b\w+ing\b`.
:::

**3. Python.** Using the iKan reviews, count the reviews that mention a driver's license (spelled *license* or *licence*, any capitalization).

:::{note} Answer
:class: dropdown
```python
text.str.contains(r"\blicen[cs]e", flags=re.IGNORECASE).sum()     # 7
```
:::

**4. Oracle SQL.** In your Oracle Cloud account, write a query that lists the SH customers whose last name contains two consecutive identical vowels (such as "aa" or "ee"), using `REGEXP_LIKE` with a group and a back-reference.

:::{note} One possible answer
:class: dropdown
```sql
SELECT cust_id, cust_first_name, cust_last_name
FROM   sh.customers
WHERE  REGEXP_LIKE(cust_last_name, '([aeiou])\1', 'i');
```
`([aeiou])` captures one vowel, and `\1` requires the same character again.
:::

## Key terms

regular expression, search expression, target string, literal, metacharacter, escape sequence, quantifier, anchor, character set, negated set, alternation, group, capture, word boundary, shorthand class, greedy and lazy matching, `re` library, `REGEXP_LIKE`, `REGEXP_SUBSTR`, `REGEXP_REPLACE`, `REGEXP_COUNT`

## Further reading

- Python documentation, [re: Regular expression operations](https://docs.python.org/3/library/re.html) and the [Regular Expression HOWTO](https://docs.python.org/3/howto/regex.html).
- Oracle, [Using regular expressions in Oracle Database](https://docs.oracle.com/en/database/oracle/oracle-database/19/adfns/regexp.html).
- Friedl, J. E. F. (2006). *Mastering Regular Expressions* (3rd ed.). O'Reilly Media.
