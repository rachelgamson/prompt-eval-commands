---
name: eval-analyze
description: Analyze prompt evaluation results — calculates metrics, names the failure pattern, and recommends one prompt change
argument-hint: [eval name, e.g. "scoring"]
allowed-tools:
  - Read
  - Write
  - AskUserQuestion
---

You are analyzing the results of a prompt evaluation. Calculate metrics, identify the dominant failure pattern, and recommend one specific prompt change to fix it.

The eval name comes from `$ARGUMENTS`. If none provided, ask for one.

---

## Step 1 — Load files

Read:
- `evals/[name]/EVAL.md`
- `evals/[name]/results.csv`

If results.csv is missing, tell the user to run `/eval-run [name]` first.

---

## Step 2 — Calculate metrics

Count all rows. Calculate:

- **Overall accuracy** = correct / total
- **Partial rate** = partial / total
- **Error rate** = (partial + incorrect) / total

If the results have sub-dimensions (e.g., usage vs infra, or multiple output fields), calculate accuracy per dimension separately. A partial result counts as wrong for the dimension it missed.

Compare overall accuracy to the pass bar defined in EVAL.md. State clearly whether the prompt passes or fails.

---

## Step 3 — Identify the failure pattern

Look at every incorrect and partial result. Ask:

- Are errors random, or do they cluster around a specific input type, score range, or condition?
- Is there a directional bias — does the prompt always err in one direction (e.g., always too low, always too conservative)?
- Is there one dimension that accounts for most of the errors?
- Do the `reason` values in results.csv point to a shared root cause?

Name the dominant failure pattern in plain language. One sentence. Be specific — not "the prompt sometimes gets it wrong" but "the prompt consistently underscores infra when the user names a tool without describing its setup."

---

## Step 4 — Write analysis

Write `evals/[name]/analysis.md`:

```
---
eval: [name]
date: [today]
prompt-version: [from EVAL.md]
---

# Eval Analysis — [eval name]

## Results

| | Count | Rate |
|---|---|---|
| Correct | N | X% |
| Partial | N | X% |
| Incorrect | N | X% |
| **Total** | N | |

Pass bar: [from EVAL.md] — **PASS / FAIL**

## Per-dimension accuracy
[table if applicable, otherwise omit]

## Dominant failure pattern
[one plain-language sentence naming exactly what is going wrong and why]

## Failure examples
[3–5 rows from results.csv that best illustrate the pattern — show id, expected, actual, reason]

## Recommended next change
[one specific, targeted change to the prompt that directly addresses the dominant failure — quote the existing text if replacing it, and show what it should become]

## What not to change
[any dimensions or behaviors that are working well — protect these from accidental edits]
```

---

## Step 5 — Surface findings inline

Show the Results table and the Dominant failure pattern and Recommended next change directly in the conversation — do not make the user open the file to see the key findings.

Then say:

> "Full report written to evals/[name]/analysis.md. After making the prompt change, re-run /eval-run [name] and compare the new accuracy to [baseline]%."

---

## Input

$ARGUMENTS
