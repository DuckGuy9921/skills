---
name: csv-data-cleaner
description: Use when the user invokes /csv-data-cleaner or asks to "clean this CSV", "profile this spreadsheet", or check a CSV or XLSX for nulls, duplicates, wrong types, outliers or encoding problems. Never overwrites the original.
---

# CSV Data Cleaner

Profile a CSV or XLSX file, apply a reproducible cleaning script that writes a new file, and report exactly what changed while leaving the original untouched.

## Steps

1. **Locate the input.** Take the file path after `/csv-data-cleaner`. If none is given, run `ls -t *.csv *.xlsx` in the working directory, use the newest match, and state that assumption. If there is no match or several equally likely ones, ask one question and stop. If the file is over 1 GB, say so and ask before profiling.

2. **Record the original fingerprint.** Run `sha256sum <input>` and keep the hash. Step 8 uses it to prove the original was not modified.

3. **Identify sheet, encoding and delimiter.** For XLSX, run `python3 -I -c 'import openpyxl,sys;print(openpyxl.load_workbook(sys.argv[1],read_only=True).sheetnames)' <input>` and use the first sheet unless the user named one. For CSV, get the charset candidate from `file -i <input>`, then pick the delimiter (comma, semicolon or tab) that gives the same field count on the first five lines. Accept the first encoding (UTF-8, then the `file -i` charset) that decodes the whole file without error.

4. **Profile every column.** Load with `pandas.read_csv(..., dtype=str, keep_default_na=False)` so raw values stay visible. For each column report its type (numeric, date, boolean or text, when at least 95% of non-empty values parse with `pd.to_numeric` or `pd.to_datetime`), null count (empty cells plus tokens such as `NA`, `N/A`, `null`, `-`), distinct count, and up to three examples.

5. **Count duplicates.** Strip whitespace and turn null tokens into empty cells first, then count fully identical rows with `df.duplicated().sum()`, so the count matches what the script removes. If a column named `id` or ending in `_id` exists, also count its repeated values. Report both numbers.

6. **Detect outliers.** For each numeric column (one that passed the 95% rule), compute Q1, Q3 and IQR, and count values outside [Q1 − 1.5·IQR, Q3 + 1.5·IQR]. Do not drop or change them; the script adds a `<col>_outlier` flag column instead.

7. **Write the cleaning script.** Save it as `<stem>_clean.py` beside the input. It reads with the detected encoding, delimiter and sheet, all as strings; strips surrounding whitespace; turns null tokens into empty cells; rewrites a column as numbers or ISO dates only if it passes the 95% rule, otherwise keeps text; drops only fully identical duplicate rows; adds the outlier flags; and writes `<stem>_cleaned.csv` beside the input. It exits with an error if the output path equals the input path.

8. **Run and verify.** Run `python3 -I <stem>_clean.py`. If an import fails, report the error rather than dropping `-I` or installing packages. Re-run `sha256sum <input>` and confirm it matches step 2. Confirm the cleaned row count equals the original count minus the duplicates removed, and the column count equals the original plus the flag columns.

9. **Report.** Use the format below, with counts and column names rather than data rows.

## Output format

```
## Input
<absolute path>, sheet (XLSX only), encoding, delimiter, rows x columns

## Profile
| Column | Type | Nulls | Distinct | Outliers | Examples |

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

- Never write to, rename or move the original. If a target name already exists, add a numeric suffix rather than replacing it.
- Do not impute missing values, drop rows with nulls or remove outliers. Report them and let the user decide.
- Keep ambiguous dates (for example `03/04/2025`, where day and month could be swapped) as text and flag them.
- If no encoding candidate parses the file, stop and report the error.
- If pandas or openpyxl is missing, report the import error. Do not install packages silently.
- For a multi-sheet workbook, list the unprocessed sheet names under "Flagged, not changed".
- Do not paste the dataset into chat.
