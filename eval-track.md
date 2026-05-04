Log eval results to the tracker — compute metrics, capture what changed, and append a row to eval-tracker.csv.

Run this after `/eval-analyze [name]`.

## Arguments

$ARGUMENTS should be the eval name (e.g. `scoring`). Leave blank to be prompted.

---

## Steps

### 1. Identify the eval

- If $ARGUMENTS is provided, use it as the eval name
- Otherwise, list subdirectories of `evals/` and ask the user to pick one
- Load `evals/[name]/EVAL.md`
- Load `evals/[name]/results.csv`
- Load `evals/[name]/analysis.md` if it exists

### 2. Compute metrics from results.csv

`results.csv` has columns: `id`, `input_file`, `expected_output`, `actual_output`, `judgment` (correct / partial / incorrect), `explanation`

Compute:
- `n_tested` — total row count
- `correct_pct` — % judged correct (integer %)
- `partial_pct` — % judged partial (integer %)
- `pass_bar` — from EVAL.md (the target accuracy % the user set during setup)
- `pass_fail` — PASS if correct_pct >= pass_bar, FAIL otherwise

### 3. Get prompt version

Read from EVAL.md if a version field is present. If not, ask: "What version label should we use for this run?" (e.g. `v1`, `v2`, `v3-tighter-rubric`)

### 4. Pull dominant failure from analysis.md

- If `analysis.md` exists: extract the one-sentence dominant failure description
- If not: set to blank and note "run /eval-analyze first for this field"

### 5. Compute delta vs previous version

- Read `evals/eval-tracker.csv` if it exists
- Find the most recent row where `skill` matches the current eval name
- Compute: current `correct_pct` minus previous `correct_pct`, shown as `+X%` or `-X%`
- If no prior row exists: set `delta_vs_prev` to `baseline`

### 6. Show computed summary and ask for input

Display a summary table:

```
Eval:             [name]
Prompt version:   [version]
Tested:           [n_tested]
Correct:          [correct_pct]%
Partial:          [partial_pct]%
Pass bar:         [pass_bar]%
Result:           [PASS / FAIL]
Delta vs prev:    [+/-X% or baseline]
Dominant failure: [from analysis.md or blank]
```

Then ask two questions:

1. **"What changed in this version?"** → `change_summary` (1–2 sentences: what was changed and why)
2. **"Anything else worth flagging for the team?"** → offer a draft of `notable_patterns`, ask to confirm or rewrite

### 7. Draft notable_patterns from data

Auto-draft a 1–2 sentence pattern note that covers:
- Whether the run passed or failed vs the pass bar
- Delta direction vs the previous version (improving, regressing, or baseline)
- The dominant failure if present
- Any inputs that share the same failure explanation (flag if >30% share one root cause)

Present this as a draft. Let the user edit before saving.

### 8. Append to eval-tracker.csv

Write to `evals/eval-tracker.csv`. Create the file with the header row if it doesn't exist yet.

Append one new row:

```
[date],[skill],[prompt_version],[change_summary],[n_tested],[correct_pct]%,[partial_pct]%,[pass_fail],[delta_vs_prev],[dominant_failure],[notable_patterns],[results_file]
```

Where `results_file` = `evals/[name]/results.csv`.

Confirm to the user: show the full appended row.

---

## Tracker schema

```
date, skill, prompt_version, change_summary, n_tested, correct_pct, partial_pct, pass_fail, delta_vs_prev, dominant_failure, notable_patterns, results_file
```

One row per eval run. The tracker lives at `evals/eval-tracker.csv` and accumulates across all evals in the project.
