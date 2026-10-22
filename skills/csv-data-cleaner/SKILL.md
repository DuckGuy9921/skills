---
name: csv-data-cleaner
description: Use when the user invokes /csv-data-cleaner or asks to "clean this CSV", "profile this spreadsheet", or check a CSV or XLSX for nulls, duplicates, wrong types, outliers or encoding problems. Never overwrites the original.
---

# CSV Data Cleaner

Profile a CSV or XLSX file, apply a reproducible cleaning script that writes a new file, and report exactly what changed while leaving the original untouched.

## Steps

1. **Locate the input.** Take the file path after `/csv-data-cleaner` and resolve it to an absolute path. If none is given, run `ls -t *.csv *.xlsx` in the working directory, use the newest match, and state that assumption. If there is no match, ask one question and stop. If the file is over 1 GB, say so and ask before profiling.

2. **Record the original fingerprint.** Run `sha256sum <input>` and keep the hash. Step 8 compares against it.

3. **Identify sheet, encoding and delimiter.** For XLSX, list the sheet names with `python3 -I`, openpyxl and `load_workbook(path, read_only=True).sheetnames`, and use the first sheet unless the user named one. For CSV, get the charset candidate from `file -i <input>`, then pick the delimiter (comma, semicolon or tab) that gives the same field count on the first five records, read with Python's `csv` module so quoted fields are respected. If none does, treat the file as one column and say so. Accept the first of UTF-8 or the `file -i` charset that decodes the whole file.

4. **Profile every column.** Load with `pandas.read_csv(..., dtype=str, keep_default_na=False)` (or `pd.read_excel` with the same options for XLSX) so raw values stay visible. For each column report its type (numeric, date, boolean or text, when at least 95% of non-empty values parse with `pd.to_numeric` or `pd.to_datetime`), null count (empty cells plus tokens such as `NA`, `N/A`, `null`, `-`), distinct count, and up to three examples.

5. **Count duplicates.** Normalize whitespace and null tokens first, then count fully identical rows with `df.duplicated().sum()`, matching what the script removes. If a column named `id` or ending in `_id` exists, also count its repeated values. Report both numbers.

6. **Detect outliers.** For each numeric column (one that passed the 95% rule), convert with `pd.to_numeric(errors="coerce")`, ignore empty cells, compute Q1, Q3 and IQR, and count values outside [Q1 − 1.5·IQR, Q3 + 1.5·IQR]. The script flags them in `<col>_outlier` columns instead.

7. **Write the cleaning script.** Save it beside the input as `<stem>_clean.py` (suffix rule in Notes). It reads with the detected encoding, delimiter and sheet, all as strings; strips surrounding whitespace; turns null tokens into empty cells; drops only fully identical duplicate rows, checked on normalized strings before type conversion, as in step 5; rewrites a column as numbers or ISO dates only if it passes the 95% rule, otherwise keeps text; keeps date columns with ambiguous day/month values as text (see Notes); adds the outlier flags; and writes `<stem>_cleaned.csv` beside the input. It exits with an error if the output path equals the input path.

8. **Run and verify.** Run `python3 -I <absolute path to the script>`, since the script sits beside the input, not necessarily in the working directory. If an import fails, report it per Notes. Re-run `sha256sum <input>` and confirm it matches step 2. Confirm the cleaned row count equals the original count minus the duplicates removed, and the column count equals the original plus the flag columns.

9. **Report.** Use the format below; no data rows.

## Output format

```
## Input
<absolute path>, sheet (XLSX only), encoding, delimiter, rows x columns

## Profile
| Column | Type | Nulls | Distinct | Outliers | Examples |
|---|---|---|---|---|---|

## Changes applied
- <change>: <count> cells or rows, <columns>

## Flagged, not changed
- <issue and count>

## Files
- Original: <absolute path>, unchanged (sha256 <hash> before and after)
- Script: <absolute path>
- Cleaned output: <absolute path>, rows x columns
```

## Notes

- Never write to, rename or move the original. Add a numeric suffix rather than replacing any existing target file.
- Do not impute missing values, drop rows with nulls or remove outliers. Report them and let the user decide.
- A date column with values whose day and month could be swapped (for example `03/04/2025`) stays text and is flagged. Only unambiguous dates become ISO dates.
- If no encoding candidate parses the file, stop and report the error.
- Report missing pandas or openpyxl; do not install packages silently.
- For a multi-sheet workbook, list the unprocessed sheet names under "Flagged, not changed".
- Do not paste the dataset into chat.
