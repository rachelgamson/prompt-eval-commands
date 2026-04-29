---
name: eval-setup
description: Set up an evaluation spec for a prompt — walks through 6 questions and writes EVAL.md + ground-truth template
argument-hint: [name for this eval, e.g. "scoring" or "suggestions"]
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
---

You are setting up a prompt evaluation spec. Walk the user through 6 questions one at a time, then write two files: a spec and an empty ground-truth template.

The eval name comes from `$ARGUMENTS`. If none provided, ask for one before starting.

---

## Step 1 — Find the prompt

Ask the user:

> "What is the path to the prompt file you're evaluating? (e.g., prototypes/v1/skills/scoring.md)"

Read that file silently. Note what it does before asking the next question.

---

## Step 2 — Ask the 6 spec questions

Ask these one at a time. Do not move to the next until the user has given a substantive answer.

**Q1 — Task**
> "In one sentence: what is this prompt supposed to do?"

**Q2 — Correct output**
> "What does a correct output look like? Describe both the format and the content it must include to be right."

**Q3 — Failure modes**
> "What are the ways this prompt gets it wrong? List the failure modes you've already seen or suspect."

**Q4 — Input variables**
> "What changes between test cases — what is the input that varies each time you run this?"

**Q5 — Judge method**
> "Who or what decides if an output is correct: you reviewing it manually, an LLM comparing it to the expected output, or both? If LLM, what criteria should it use?"

**Q6 — Pass bar**
> "What accuracy rate makes this prompt good enough — what's the number you're aiming for? (e.g., 85% agreement with human labels on the key dimension)"

---

## Step 3 — Confirm before writing

Show a summary of the 6 answers and ask:

> "Does this look right, or anything to adjust before I write the files?"

Apply any changes before proceeding.

---

## Step 4 — Write the files

Determine the output directory: `evals/$ARGUMENTS/` relative to the current working directory.

**Write `evals/[name]/EVAL.md`:**

```
---
prompt: [path from Step 1]
created: [today's date]
version: 1
---

# Eval Spec — [eval name]

## Task
[Q1 answer]

## Correct output
[Q2 answer]

## Failure modes
[Q3 answer]

## Input variables
[Q4 answer]

## Judge method
[Q5 answer]

## Pass bar
[Q6 answer]

## Ground truth file
evals/[name]/ground-truth.csv

## Results file
evals/[name]/results.csv

## Judgment rules
- correct = output matches expected on all key dimensions
- partial = output is directionally right but off on one dimension
- incorrect = output is wrong on the primary dimension
```

**Write `evals/[name]/ground-truth.csv`** with headers only:

```
id,input_file,expected_output,notes
```

Where:
- `input_file` = path to the input document (e.g., path to a transcript .md file)
- `expected_output` = the correct output for this input (format matches what the prompt returns)
- `notes` = brief explanation of why this is the correct answer — key for understanding edge cases

---

## Step 5 — Tell the user what to do next

Say:

> "Spec written. Two files created:
> - `evals/[name]/EVAL.md` — your eval spec
> - `evals/[name]/ground-truth.csv` — fill this in with your labeled examples
>
> For each row in ground-truth.csv: add the path to the input file, the correct expected output, and a short note explaining why it's correct.
>
> When ready, run `/eval-run [name]` to score your ground truth set against the current prompt."

---

## Input

$ARGUMENTS
