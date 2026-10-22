---
name: code-explainer
description: Use when the user invokes /code-explainer or asks to explain unfamiliar code, a file, or a function, e.g. "how does this work", "walk me through X", "trace the entry point", "what does this module do".
---

# Code Explainer

Explain an unfamiliar file, function, or module by tracing its entry points, data flow, and invariants, citing real file and line locations.

## Steps

1. **Get the target.** Use the text after `/code-explainer` as the target: a file path, a function or class name, or a `path:line`. If there is none, use the code most recently discussed in the conversation. If that is also missing, ask one question naming the file or symbol you need, then stop. Pitch the explanation at a working developer who has not read this code.

2. **Locate the definition.** For a path, run `Read` on the file. For a symbol, run `Grep` with `output_mode: "content"` and `-n: true` using a pattern such as `def NAME|function NAME|class NAME|NAME\s*=` across the repo. If several definitions match, explain the most likely one and list the others. Record each definition as `/abs/path/file.ext:line`.

3. **Find entry points.** Grep for call sites of the symbol, excluding its definition. Check how it is reached: exported or imported names, route or handler registration, a CLI `main` or `if __name__ == "__main__"` block, `package.json` `scripts` or `bin`, and event or scheduled hooks. Trace callers up one or two levels until you reach something a user or system triggers.

4. **Trace the data flow.** Read the full function body. List its inputs and where each comes from, each transformation in order, its outputs, and its side effects (file, network, database, global state, logging). Read the callees that affect the result. Skip utility code that does not change the answer.

5. **Identify invariants.** Look for guard clauses, `assert` or `raise` statements, type hints, validation, and comments that state assumptions. Grep the test directories for the symbol to find tests that pin behavior. Tag each invariant `enforced` (code checks it), `tested` (only a test pins it), or `assumed` (neither; say so).

6. **Check history when intent is unclear.** If a purpose or a surprising branch is not explained by the code or tests, run `git log --oneline -5 -- <file>`, and for a specific line `git blame -L <start>,<end> <file>`. Quote a commit message only when it supports your reading.

7. **Verify before writing.** Re-read every `file:line` you plan to cite and confirm it says what you claim. Do not describe behavior you have not read. Label anything you inferred as inferred.

8. **Parallelise wide traces.** If the trace spans many modules, spawn one subagent per module with `model: haiku` to read and summarise its part, then combine the results yourself. Skip this for a single file.

## Output format

**Summary:** one or two sentences on what the code does and who calls it.

**Entry points:** bullets, each as `/abs/path/file.ext:line` followed by what triggers it.

**Data flow:** numbered steps from input to output, each with a `file:line` reference.

**Key invariants:** bullets, each tagged `enforced`, `tested`, or `assumed`.

**Read next:** up to three files or symbols worth reading to go deeper.

## Notes

- Use read-only tools. Do not modify files or run the code unless the user asks.
- Do not paste large code blocks. Quote one line only when its exact text matters, such as a surprising condition.
- Do not write report or markdown files; return the explanation in the reply.
- If the target does not exist, say so and offer the closest Grep match instead of guessing.
- Do not claim a bug unless you can show the code path that produces it. Report suspicions separately as "possible issue".
- Keep a single-function explanation under about 500 words. Go longer for a module only if the user asks for depth.
