# Shape-Aware Fallbacks and Detective Report Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add inline fallback hints to four notebook task cells so all 15 datasets work without TA intervention; replace the Detective Report cell with a 6-prompt synthesis exercise; add the cheatsheet entries the new fallbacks rely on.

**Architecture:** Three independent groups of edits — (1) append/insert short comment blocks into four existing code cells of `notebook.ipynb`, (2) replace the source of the Detective Report markdown cell, (3) add two student cheatsheet bullets and update two TA cheatsheet per-dataset entries. Each task in the plan produces one self-contained commit.

**Tech Stack:** Jupyter notebook (ipynb JSON, edited via `NotebookEdit`), Markdown.

---

## Spec

Source spec: [2026-04-25-shape-aware-fallbacks-and-detective-report-design.md](../specs/2026-04-25-shape-aware-fallbacks-and-detective-report-design.md)

This plan implements that spec exactly. Cell IDs (verified at plan time):

| Index | Cell ID | Task |
| --- | --- | --- |
| 0 | `d66ddc83` | Task 1 (Load) |
| 1 | `af606d09` | Task 2 (Head) |
| 2 | `3871cee3` | Task 3 (Missing) |
| 3 | `564f921a` | Task 4 (describe) — modified by Task 1 of plan |
| 4 | `08c10222` | Task 5 (value_counts) |
| 5 | `2bc1092b` | Task 6 (Sort & filter) — modified by Task 2 of plan |
| 6 | `1b0312dc` | Task 7 (Derive) — modified by Task 3 of plan |
| 7 | `c11cb1b4` | Task 8 (Group and average) — modified by Task 4 of plan |
| 8 | `a1e4b985` | Task 9 (Plot) |
| 9 | `88fd3022` | Detective Report (markdown) — modified by Task 5 of plan |

---

## File Structure

| Path | Action |
| --- | --- |
| `content/notebook.ipynb` | modify cells `564f921a`, `2bc1092b`, `1b0312dc`, `c11cb1b4`, `88fd3022` |
| `content/student_cheatsheet.md` | append one bullet to "Building" section, one bullet to "Analyzing" section |
| `content/ta_cheatsheet.md` | replace categorical-suggestion line in Dataset 4 and Dataset 7 entries |

No other files are touched.

---

### Task 1: Append fallback hint to Task 4 cell

**Files:**
- Modify: `content/notebook.ipynb` cell `564f921a`

**Background:** survey-shaped datasets (#3 Thanksgiving, #5 Weather) have only `RespondentID` as a numeric column, so `describe()` returns thin output. We add a fallback note pointing them forward to Tasks 5 and 8.

- [ ] **Step 1: Replace cell `564f921a` source**

Use the `NotebookEdit` tool with:
- `notebook_path`: `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/notebook.ipynb`
- `cell_id`: `564f921a`
- `edit_mode`: `replace`
- `cell_type`: `code`
- `new_source`:

```
# TASK 4 — Generate a statistical summary
#
# Now you want a quick numerical fingerprint of your dataset:
# the mean, the standard deviation, the min and max of every
# numeric column. There is a single pandas method that gives you
# all of these in one table.
#
# Your job:
#   - Generate the descriptive statistics for `df`.
#
# Then look at the output and discuss with your team:
#   - Are the values in the hundreds? Thousands? Decimals?
#   - What does the std (standard deviation) tell you about how
#     consistent or spread-out the values are?
#   - Do any columns look surprisingly small, large, or noisy?
#
# Check the cheatsheet under "Analyzing" for the right function.
#
# Note: if your dataset has only one numeric column (or none),
# you'll get a thin describe() output — that's OK. Your insights
# will come from Tasks 5 and 8 instead.
```

- [ ] **Step 2: Verify the cell content matches**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
src = ''.join(nb['cells'][3]['source'])
assert 'thin describe() output' in src, 'fallback not present'
assert src.startswith('# TASK 4 — Generate a statistical summary'), 'heading changed'
assert src.endswith('Tasks 5 and 8 instead.'), 'unexpected ending'
print('ok')
"
```
Expected output: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "add survey-dataset fallback to notebook task 4"
```

---

### Task 2: Append fallback hint to Task 6 cell

**Files:**
- Modify: `content/notebook.ipynb` cell `2bc1092b`

**Background:** survey-shaped datasets have no useful numeric column to sort on. The fallback tells those teams to skip Task 6 with confidence.

- [ ] **Step 1: Replace cell `2bc1092b` source**

Use `NotebookEdit` with `cell_id=2bc1092b`, `edit_mode=replace`, `cell_type=code`, `new_source`:

```
# TASK 6 — Sort and filter
#
# Now you'll start asking questions of your data. There are two
# sub-tasks here:
#
# Sub-task A — Top 5:
#   - Pick a numeric column from your dataset (you saw the column
#     names in Task 2).
#   - Sort `df` by that column from highest to lowest.
#   - Show only the top 5 rows.
#
# Sub-task B — Filter:
#   - Pick a numeric column (the same one or a different one).
#   - Find all the rows where that column's value is greater than
#     the column's average (mean).
#
# Check the cheatsheet under "Analyzing" for sorting, filtering,
# and computing the mean.
#
# Note: if your dataset has no useful numeric column to sort or
# filter on, skip this task and move on — Tasks 7, 8, and 9 will
# still give your team plenty to do.
```

- [ ] **Step 2: Verify**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
src = ''.join(nb['cells'][5]['source'])
assert 'skip this task and move on' in src
assert src.startswith('# TASK 6 — Sort and filter')
assert 'Sub-task A' in src and 'Sub-task B' in src
print('ok')
"
```
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "add survey-dataset fallback to notebook task 6"
```

---

### Task 3: Insert boolean-derive branch in Task 7 cell

**Files:**
- Modify: `content/notebook.ipynb` cell `1b0312dc`

**Background:** unique-rows datasets (#4 Unisex names, #7 Alcohol) lack a usable categorical column. Survey datasets (#3, #5) lack two numeric columns to combine. Both need an alternative derive recipe — a True/False category column derived from a single column. We insert this as a branch in the existing examples block.

- [ ] **Step 1: Replace cell `1b0312dc` source**

Use `NotebookEdit` with `cell_id=1b0312dc`, `edit_mode=replace`, `cell_type=code`, `new_source`:

```
# TASK 7 — Derive a new column
#
# Sometimes the column you want to analyze isn't already in the dataset
# — you have to build it yourself. For example, if you have separate
# `beer_servings` and `wine_servings` columns, you might want a
# `total_servings` column that adds them together.
#
# Your job:
#   - Pick one or two existing numeric columns from `df`.
#   - Create a NEW column whose values are computed from those existing
#     column(s). Examples:
#       - a sum:        df["total"] = df["a"] + df["b"]
#       - a difference: df["diff"]  = df["a"] - df["b"]
#       - a ratio:      df["ratio"] = df["a"] / df["b"]
#   - Or, if your dataset doesn't have two numeric columns to combine,
#     derive a True/False category from a single column instead:
#       - boolean:      df["new_cat"] = df["a"] > value
#     This gives you a fresh category column you can use in Tasks 8 and 9.
#   - Confirm it worked by running `df.head()` again — your new column
#     should appear at the right of the table.
#
# Check the cheatsheet under "Building" for the syntax pattern.
```

- [ ] **Step 2: Verify**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
src = ''.join(nb['cells'][6]['source'])
assert 'True/False category from a single column' in src
assert 'df[\"new_cat\"] = df[\"a\"] > value' in src
assert src.startswith('# TASK 7 — Derive a new column')
assert 'Confirm it worked by running' in src
print('ok')
"
```
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "add boolean-derive fallback to notebook task 7"
```

---

### Task 4: Append `.size()` fallback to Task 8 cell

**Files:**
- Modify: `content/notebook.ipynb` cell `c11cb1b4`

**Background:** for survey datasets, `.mean()` doesn't make sense (the only numeric column is an ID). The fallback shows them `.size()` as the alternative.

- [ ] **Step 1: Replace cell `c11cb1b4` source**

Use `NotebookEdit` with `cell_id=c11cb1b4`, `edit_mode=replace`, `cell_type=code`, `new_source`:

```
# TASK 8 — Group and average
#
# Now you'll combine a categorical column with a numeric column to find
# which categories have the highest or lowest averages.
#
# Your job:
#   - Pick a categorical column from `df` (any column whose values are
#     categories, words, or labels rather than numbers).
#   - Pick a numeric column from `df`.
#   - Compute the AVERAGE (mean) of the numeric column for each
#     category in the categorical column.
#
# Then look at the result and discuss with your team:
#   - Which category has the highest average?
#   - Which has the lowest?
#   - Anything unexpected?
#
# Check the cheatsheet under "Analyzing" for the function you need.
#
# Note: if .mean() doesn't make sense for your data (for example, your
# only numeric column is an ID), use .size() instead — it counts how
# many rows are in each group, which is just as interesting:
#       df.groupby("cat_col").size()
```

- [ ] **Step 2: Verify**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
src = ''.join(nb['cells'][7]['source'])
assert '.size() instead' in src
assert 'df.groupby(\"cat_col\").size()' in src
assert src.startswith('# TASK 8 — Group and average')
print('ok')
"
```
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "add .size() fallback to notebook task 8"
```

---

### Task 5: Replace Detective Report cell content

**Files:**
- Modify: `content/notebook.ipynb` cell `88fd3022`

**Background:** the existing 4-prompt report assumes "average value" exists and uses an outdated "El Paso weather" framing. The new 6-prompt version is dataset-shape-agnostic, rewards work done in the new tasks, and ends with a synthesis ("team headline") prompt. Cell type stays markdown.

- [ ] **Step 1: Replace cell `88fd3022` source**

Use `NotebookEdit` with `cell_id=88fd3022`, `edit_mode=replace`, `cell_type=markdown`, `new_source`:

```
### Team Detective Report

Now that you've explored your dataset, fill out this report with your team. Each prompt asks about something you produced in the tasks above — go back to the relevant cell to grab numbers or column names if you need to.

1. **The dataset.** What is your data about? Which of your columns made it obvious?

2. **The most surprising number** (from Task 4 or Task 8). Pick one number from `describe()` or `groupby` that surprised you. What was it, and why?

3. **The biggest split** (from Task 5 or Task 8). Which value or group came out on top — the most common category, or the highest/lowest group average?

4. **The strongest pattern in your plot** (from Task 9). What does your chart show — a trend, a difference between groups, an outlier? One sentence.

5. **One question your dataset CAN'T answer.** What did you wish you could analyze, but the columns you have don't support?

6. **Your team's headline.** Write a single sentence that captures the most interesting thing you found, as if it were a news article headline.
```

- [ ] **Step 2: Verify cell type and content**

Use a python heredoc (single-quoted to avoid shell-escaping the apostrophes in `CAN'T` and `team's`):

```bash
python3 <<'PY'
import json, re
nb = json.load(open('content/notebook.ipynb'))
c = nb['cells'][9]
assert c['cell_type'] == 'markdown', f"cell type is {c['cell_type']}, expected markdown"
src = ''.join(c['source'])
assert src.startswith('### Team Detective Report')
for label in ['The dataset', 'The most surprising number', 'The biggest split',
              'The strongest pattern', "CAN'T answer", "team's headline"]:
    assert label in src, f'missing prompt label: {label!r}'
nums = re.findall(r'^\d+\.', src, re.MULTILINE)
assert nums == ['1.', '2.', '3.', '4.', '5.', '6.'], f'numbered prompts mismatch: {nums}'
assert 'El Paso' not in src
assert 'average value was' not in src
print('ok')
PY
```
Expected: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "replace detective report with 6-prompt synthesis exercise"
```

---

### Task 6: Add `.size()` and boolean-derive bullets to student cheatsheet

**Files:**
- Modify: `content/student_cheatsheet.md`

**Background:** the new fallbacks reference two patterns students need to look up: `df.groupby("cat_col").size()` (Task 8 fallback) and `df["new_col"] = df["a"] > value` (Task 7 fallback). We add one bullet to each existing section ("Building" and "Analyzing"). No section reordering, no other changes.

- [ ] **Step 1: Add the boolean-derive bullet to "Building"**

Use the `Edit` tool with file path `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/student_cheatsheet.md`.

`old_string`:
```
- `df["new_col"] = df["a"] / df["b"]` — create a new column that's a ratio.

## Analyzing
```

`new_string`:
```
- `df["new_col"] = df["a"] / df["b"]` — create a new column that's a ratio.
- `df["new_col"] = df["a"] > value` — create a True/False column you can use as a category for value_counts or groupby.

## Analyzing
```

- [ ] **Step 2: Add the `.size()` bullet to "Analyzing"**

Use the `Edit` tool with the same file path.

`old_string`:
```
- `df.groupby("cat_col")["num_col"].mean()` — average of `num_col` for each category in `cat_col`.

## Plotting
```

`new_string`:
```
- `df.groupby("cat_col")["num_col"].mean()` — average of `num_col` for each category in `cat_col`.
- `df.groupby("cat_col").size()` — count how many rows are in each category (alternative to `.mean()` when there's no useful numeric column).

## Plotting
```

- [ ] **Step 3: Verify**

```bash
grep -c '^- ' content/student_cheatsheet.md
grep -E '^## ' content/student_cheatsheet.md
grep -c 'value_counts\|groupby\|> value' content/student_cheatsheet.md
python3 -c "
text = open('content/student_cheatsheet.md').read()
assert 'df[\"new_col\"] = df[\"a\"] > value' in text, 'boolean-derive missing'
assert 'df.groupby(\"cat_col\").size()' in text, '.size() missing'
# section order unchanged
import re
sections = re.findall(r'^## (.+)$', text, re.MULTILINE)
assert sections == ['Loading data','Inspecting','Cleaning','Building','Analyzing','Plotting'], sections
print('ok')
"
```
Expected: 6 sections in order, last python check prints `ok`.

- [ ] **Step 4: Commit**

```bash
git add content/student_cheatsheet.md
git commit -m "add .size() and boolean-derive entries to student cheatsheet"
```

---

### Task 7: Update TA cheatsheet — Datasets 4 and 7 categorical lines

**Files:**
- Modify: `content/ta_cheatsheet.md`

**Background:** datasets #4 (unisex names) and #7 (alcohol) have no usable categorical column. The boolean-derive recipe in the notebook fallback gives those teams a path. We update each per-dataset entry's "categorical:" line under **Suggested columns** to cite the recipe directly. Quirks bullets are unchanged.

- [ ] **Step 1: Update Dataset 4 categorical line**

Use the `Edit` tool with file path `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/ta_cheatsheet.md`.

`old_string`:
```
- categorical: `name` is technically categorical but each is unique; tell them this dataset has no good column for `value_counts` and they'll get most insight from sort/filter and plot.
```

`new_string`:
```
- categorical: `name` is unique-per-row, so this team needs to derive a category. Show them the boolean-derive recipe: `df["male_dominant"] = df["male_share"] > df["female_share"]`. Use the new column for value_counts and groupby.
```

- [ ] **Step 2: Update Dataset 7 categorical line**

Use the `Edit` tool with the same file path.

`old_string`:
```
- categorical: `country` is unique-per-row.
```

`new_string`:
```
- categorical: `country` is unique-per-row, so this team needs to derive a category. Show them the boolean-derive recipe: `df["heavy_drinker"] = df["total_litres_of_pure_alcohol"] > df["total_litres_of_pure_alcohol"].median()`. Use the new column for value_counts and groupby.
```

- [ ] **Step 3: Verify both edits**

```bash
python3 -c "
text = open('content/ta_cheatsheet.md').read()
assert 'df[\"male_dominant\"] = df[\"male_share\"] > df[\"female_share\"]' in text, 'd4 recipe missing'
assert 'df[\"heavy_drinker\"] = df[\"total_litres_of_pure_alcohol\"]' in text, 'd7 recipe missing'
# old lines should be gone
assert 'tell them this dataset has no good column for' not in text, 'old d4 line still present'
# count dataset entries unchanged
import re
ds = re.findall(r'^### Dataset \d+', text, re.MULTILINE)
assert len(ds) == 15, f'dataset count {len(ds)}'
print('ok')
"
```
Expected: `ok`

- [ ] **Step 4: Commit**

```bash
git add content/ta_cheatsheet.md
git commit -m "update TA cheatsheet datasets 4 and 7 with boolean-derive recipe"
```

---

### Task 8: Final cross-check (verification only — no commit)

**Files:** none modified.

- [ ] **Step 1: Notebook integrity**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
assert len(nb['cells']) == 10
heads = [''.join(c['source']).splitlines()[0] for c in nb['cells']]
expected = [
    'import pandas as pd',
    '# TASK 2 — Look at the first few rows of your dataset',
    '# TASK 3 — Find and handle missing values',
    '# TASK 4 — Generate a statistical summary',
    '# TASK 5 — Count categories',
    '# TASK 6 — Sort and filter',
    '# TASK 7 — Derive a new column',
    '# TASK 8 — Group and average',
    '# TASK 9 — Plot a relationship between two columns',
    '### Team Detective Report',
]
for i,(g,w) in enumerate(zip(heads, expected)):
    assert g == w, f'cell {i}: got {g!r}'
# verify Detective Report is still markdown
assert nb['cells'][9]['cell_type'] == 'markdown'
# verify each fallback hit its target cell
checks = {
    3: 'thin describe() output',
    5: 'skip this task and move on',
    6: 'True/False category from a single column',
    7: '.size() instead',
}
for i, needle in checks.items():
    src = ''.join(nb['cells'][i]['source'])
    assert needle in src, f'cell {i} missing fallback {needle!r}'
print('notebook ok')
"
```
Expected: `notebook ok`

- [ ] **Step 2: Cheatsheet integrity**

```bash
python3 -c "
import re
text = open('content/student_cheatsheet.md').read()
sections = re.findall(r'^## (.+)$', text, re.MULTILINE)
assert sections == ['Loading data','Inspecting','Cleaning','Building','Analyzing','Plotting'], sections
assert 'df[\"new_col\"] = df[\"a\"] > value' in text
assert 'df.groupby(\"cat_col\").size()' in text
print('student cheatsheet ok')

text = open('content/ta_cheatsheet.md').read()
tasks = re.findall(r'^## Task ', text, re.MULTILINE)
assert len(tasks) == 9
ds = re.findall(r'^### Dataset \d+', text, re.MULTILINE)
assert len(ds) == 15
assert 'male_dominant' in text and 'heavy_drinker' in text
print('ta cheatsheet ok')
"
```
Expected: two `ok` lines.

- [ ] **Step 3: Commit history is clean (7 commits this plan, in order)**

```bash
git log --oneline 267c409..HEAD
```
Expected: 7 commits with descriptive messages — `add survey-dataset fallback to notebook task 4`, `add survey-dataset fallback to notebook task 6`, `add boolean-derive fallback to notebook task 7`, `add .size() fallback to notebook task 8`, `replace detective report with 6-prompt synthesis exercise`, `add .size() and boolean-derive entries to student cheatsheet`, `update TA cheatsheet datasets 4 and 7 with boolean-derive recipe`.

- [ ] **Step 4: No commit needed for verification-only task.**
