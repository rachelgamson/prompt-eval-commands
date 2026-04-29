---
name: eval-run
description: Run a prompt evaluation against your ground truth dataset — scores each example and writes results.csv
argument-hint: [eval name, e.g. "scoring"]
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

You are running a prompt evaluation. Read the spec, load the ground truth, apply the prompt to each example, judge each output, and write results.

The eval name comes from `$ARGUMENTS`. If none provided, ask for one.

---

## Step 1 — Load the spec

Read `evals/[name]/EVAL.md` and `evals/[name]/ground-truth.csv`.

If either file is missing, stop and tell the user: "Run /eval-setup [name] first."

---

## Step 2 — Validate ground truth

Count the filled rows in ground-truth.csv (rows where input_file, expected_output, and notes are all present).

If fewer than 12 filled rows, tell the user:

> "You have [N] labeled examples. 12 is the recommended minimum for a reliable baseline. Continue anyway? (yes / no)"

Wait for confirmation before proceeding.

---

## Step 3 — Load the prompt

Read the prompt file referenced in EVAL.md. This is the prompt you will apply to each input.

---

## Step 4 — Score each example

For each row in ground-truth.csv:

1. Read the input file at the path in `input_file`
2. Apply the prompt to that input — generate the output as the prompt would
3. Compare your output to the `expected_output` in the row
4. Assign a judgment using the judgment rules from EVAL.md:
   - **correct** — output matches expected on all key dimensions
   - **partial** — directionally right but off on one dimension
   - **incorrect** — wrong on the primary dimension
5. Write a reason in 10 words or fewer explaining the judgment

Do this for every row before writing results. Do not stop mid-run.

---

## Step 5 — Write results

Write `evals/[name]/results.csv` with these columns:

```
id,input_file,expected_output,actual_output,judgment,reason
```

Include every row from ground-truth.csv. Do not skip rows where the judgment was correct — those are equally important for the metrics.

---

## Step 6 — Report summary

Show the user a quick summary before pointing to the file:

> "Run complete. [N] examples scored.
> - Correct: [N] ([X]%)
> - Partial: [N] ([X]%)
> - Incorrect: [N] ([X]%)
>
> Results written to evals/[name]/results.csv.
> Run /eval-analyze [name] to see failure patterns and a recommended fix."

---

## Input

$ARGUMENTS
