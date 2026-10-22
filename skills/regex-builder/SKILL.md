---
name: regex-builder
description: Use when the user invokes /regex-builder or asks to build, write, or fix a regular expression from example strings, e.g. "regex that matches these emails", "regex for dates", "why does my regex match this", or lists of matches and non-matches.
---

# Regex Builder

Build a regular expression from concrete match and non-match samples, verify it with a real test run, and explain each part of it.

## Steps

1. **Collect samples.** Take the example strings from the request and split them into must-match and must-not-match lists. Aim for at least three of each. If the user gave only positives, add obvious negatives (empty string, wrong length, wrong character class) and state them as assumptions. Ask one clarifying question only when no examples are given and the pattern is unknown.

2. **Pick the flavor.** Use the language or tool the user names (Python `re`, JavaScript, PCRE, Go RE2, `grep -E`). If none is named, use the dominant language among the files listed by `git ls-files` in the working directory. If there is no repo or the result is inconclusive, use Python `re`. Say which flavor was chosen. RE2 has no lookaround or backreferences, so avoid those for it.

3. **Draft the simplest anchored core.** Write one pattern that matches every positive. Anchor it with `^...$` (or `\A...\Z` in Python, `\A...\z` in RE2) unless the user wants substring matching, so it also works with `search` or `grep`. Prefer explicit classes such as `[A-Za-z0-9._%+-]+` over `.*`.

4. **Build incrementally.** Add one constraint at a time, such as a length bound, a required separator, or a fixed suffix. After each change, rerun the same test script from step 5 (do not write a new one). Stop when every positive matches and every negative is rejected. Do not add special-case alternations for individual samples unless the user asks. If none fits without them, stop and report the conflicting samples.

5. **Test with a script.** Write one small test script in the scratchpad directory, or in a new `mktemp -d` directory if none is given. Never write it in the repo. It holds both sample lists, calls `re.fullmatch` (or the flavor's full-match equivalent) on each, and prints PASS or FAIL per sample. Run it with the interpreter for the chosen flavor, such as `python3 -I <script>` or `node <script>`. Use `search` only when substring matching was requested.

6. **Check for catastrophic backtracking.** Look for nested quantifiers such as `(a+)+` or `(\w+\s?)*`, alternations with overlapping branches under a quantifier such as `(a|a)*`, and adjacent unbounded quantifiers over the same class such as `\d+\d+`. Rewrite any hit to remove the ambiguity, for example `(\w+)*` becomes `\w+`. Then time the pattern on a hostile input, such as repeats of `a` followed by `!`, at two or three input lengths (for example 20, 25, 30). Run each timing under a time limit (for example `timeout 20`) to avoid hangs. Confirm the runtime grows roughly in proportion to length (about 1.5x from 20 to 30), not by a fixed factor for each added character.

7. **Explain each part.** Break the final pattern into tokens and give each one a table row: anchors, character classes, quantifiers, groups, and lookarounds.

8. **Show usage.** Give a short snippet in the target language that applies the pattern and extracts any capture groups the user needs.

## Output format

**Pattern:** `<final regex>`
**Flavor:** <language or engine>, flags: <flags or none>

**Test results:** <matched> of <total> positives matched, <rejected> of <total> negatives rejected (from the script run).

| Part | Meaning |
|---|---|
| `<token>` | <what it matches> |

**Backtracking:** <safe, or the rewrite made and the timing result>

**Usage:**
```<lang>
<snippet>
```

**Assumptions:** <bulleted list, or "none">

## Notes

- Never present a pattern as working without a test run that printed results.
- Warn the user about catastrophic backtracking whenever step 6 found a risk, even after a fix, and name the input length at which it became slow, or say it stayed fast up to the longest tested length.
- Regexes do not parse nested structures such as HTML or JSON, and full RFC 5322 email validation is impractical. Say so and suggest a parser or a looser check when the request needs that.
- If a sample appears in both lists, stop and report the contradiction. Ask one question about which side is correct.
- Do not commit the test script or modify project files unless the user asks.
- One line per token is enough.
