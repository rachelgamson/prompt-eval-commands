# Prompt Eval Commands

Four Claude Code slash commands for evaluating and improving AI prompts. Based on the ML evaluation framework: ground truth, LLM-as-judge, human review, metrics, iterate.

Use these whenever you want to know if a prompt is working — and where to fix it.

---

## Commands

### `/eval-setup [name]`
Set up an evaluation spec for a prompt. Walks you through 6 questions and writes two files:
- `evals/[name]/EVAL.md` — the spec (task, correct output, failure modes, judge method, pass bar)
- `evals/[name]/ground-truth.csv` — empty template to fill with labeled examples

Run once per prompt you want to evaluate.

### `/eval-run [name]`
Score your ground truth examples against the current prompt. Reads your spec and labeled examples, applies the prompt to each input, judges each output as correct / partial / incorrect, and writes:
- `evals/[name]/results.csv`

Run every time you change the prompt.

### `/eval-analyze [name]`
Analyze the results. Calculates accuracy metrics, identifies the dominant failure pattern, and recommends one targeted prompt change. Writes:
- `evals/[name]/analysis.md`

Shows key findings inline so you don't have to open the file.

### `/eval-track [name]`
Log the run to the tracker. Reads from `results.csv`, `EVAL.md`, and `analysis.md` to compute metrics, then asks you what changed and what's worth flagging. Appends one row to:
- `evals/eval-tracker.csv`

The tracker accumulates across all runs and versions so you can see what's improving, what's regressing, and what failure patterns keep showing up.

---

## The loop

```
/eval-setup [name]          # answer 6 questions, get the spec
# fill in ground-truth.csv  # add your labeled examples (12 minimum)
/eval-run [name]            # score everything
/eval-analyze [name]        # see what's wrong and what to fix
/eval-track [name]          # log the run to the tracker
# update the prompt
/eval-run [name]            # re-run
/eval-analyze [name]        # compare to baseline
/eval-track [name]          # log again — delta_vs_prev shows the trend
```

---

## How to install

**On your own machine (global — works in any project):**
```
cp eval-setup.md eval-run.md eval-analyze.md eval-track.md ~/.claude/commands/
```

**In a specific project (shared with anyone who clones the repo):**
```
mkdir -p your-project/.claude/commands
cp eval-setup.md eval-run.md eval-analyze.md eval-track.md your-project/.claude/commands/
```

---

## Output file structure

```
evals/
  eval-tracker.csv           ← all runs across all evals (created by /eval-track)
  [name]/
    EVAL.md                  ← spec (created by /eval-setup)
    ground-truth.csv         ← your labeled examples (you fill this in)
    results.csv              ← scoring run output (created by /eval-run)
    analysis.md              ← findings + recommendation (created by /eval-analyze)
```

---

## Tracker schema

`eval-tracker.csv` has one row per eval run:

| Column | What it captures |
|---|---|
| `date` | When the run happened |
| `skill` | Which prompt was evaluated (e.g. `scoring`, `conversation`) |
| `prompt_version` | Version label (e.g. `v1`, `v3-tighter-rubric`) |
| `change_summary` | 1–2 sentences: what changed and why |
| `n_tested` | Number of examples scored |
| `correct_pct` | % judged correct |
| `partial_pct` | % judged partial |
| `pass_fail` | PASS / FAIL vs the pass bar set in EVAL.md |
| `delta_vs_prev` | Change in correct_pct vs the prior version of the same skill |
| `dominant_failure` | One-sentence root cause from eval-analyze |
| `notable_patterns` | Human-written: anything worth flagging for the team |
| `results_file` | Path to the detailed results CSV |

---

## What you need to fill in

`ground-truth.csv` columns:

| Column | What to put |
|---|---|
| `id` | Row number |
| `input_file` | Path to the input document (e.g., a transcript .md file) |
| `expected_output` | The correct output — format matches what your prompt returns |
| `notes` | Short explanation of why this is the correct answer |

The notes column is important — it's what makes the ground truth useful when you revisit it later or hand it off to someone else.

---

## How many examples do you need?

- **12** — enough to identify a failure pattern and establish a directional baseline
- **25** — tight enough (±15% margin of error) to trust comparisons between prompt versions
- **All of them** — if your dataset is small (under 50), just label everything

---

## Requirements

- [Claude Code](https://claude.ai/code) installed
- Commands placed in `~/.claude/commands/` (global) or `.claude/commands/` (project)
