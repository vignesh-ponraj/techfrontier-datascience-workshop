# Workshop Notebook Restructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert `content/notebook.ipynb` into a task-only worksheet and add two markdown cheatsheets — one student reference, one TA guide — to support a Data Science workshop.

**Architecture:** This is a docs/content change with three independent file deliverables. No runtime code is added. Verification is by (a) re-reading each modified file to confirm content, and (b) for the notebook, validating that the JSON still parses and that cell count and ordering are preserved.

**Tech Stack:** Jupyter notebook (ipynb JSON), Markdown. Existing imports in the notebook target pandas, numpy, matplotlib, seaborn — these are already pinned in `requirements.txt` and are not modified by this plan.

---

## Spec

Source spec: [2026-04-24-workshop-notebook-restructure-design.md](../specs/2026-04-24-workshop-notebook-restructure-design.md)

Refer to the spec for the strip mapping table, cheatsheet section lists, and out-of-scope items. This plan implements the spec exactly.

---

## File Structure

| Path | Action | Responsibility |
| --- | --- | --- |
| `content/notebook.ipynb` | modify | Task-only worksheet. 7 cells preserved. Cell 1 keeps imports, loses load line, gains expanded comments. Cells 2–6 lose all Python, keep expanded comments. Cell 7 unchanged. |
| `content/student_cheatsheet.md` | create | One-line-per-function reference grouped into 5 sections (Loading, Inspecting, Cleaning, Analyzing, Plotting). |
| `content/ta_cheatsheet.md` | create | Per-task guidance (Tasks 1–6) with three subsections each: Answer, Common pitfalls, Hints to give. Preamble notes that each team has a different secret dataset. |

Each task below produces a self-contained, committable change.

---

### Task 1: Strip and expand notebook cell 1 (imports + load instructions)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `d66ddc83`

The current cell has 4 import lines, then 4 comment lines, then `df = pd.read_csv("FILENAME.csv")`. We keep the imports, **drop the `df = pd.read_csv(...)` line** (loading is now a student task), and replace the comments with a more descriptive task block. A blank line separates imports from the task block.

- [ ] **Step 1: Replace cell `d66ddc83` source**

Use the NotebookEdit tool with `notebook_path=/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/notebook.ipynb`, `cell_id=d66ddc83`, `edit_mode=replace`, `cell_type=code`, and the new source:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# TASK 1 — Load your team's dataset
#
# Each team has been assigned a different file named like
# 'secret_dataset_3.csv'. Look in the file browser on the left and
# find the one assigned to your team.
#
# Your job:
#   - Use pandas to read that CSV file.
#   - Assign the result to a variable called `df` so the rest of the
#     notebook can use it.
#
# Check the cheatsheet under "Loading data" for the function you need.
```

- [ ] **Step 2: Verify the notebook JSON still parses and cell count is unchanged**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); print(len(nb['cells']))"`
Expected output: `7`

- [ ] **Step 3: Verify cell 1 no longer contains `pd.read_csv`**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][0]['source']); assert 'pd.read_csv' not in src, 'load line still present'; assert 'import pandas' in src, 'imports were removed'; print('ok')"`
Expected output: `ok`

- [ ] **Step 4: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip load line from notebook cell 1, expand task description"
```

---

### Task 2: Strip notebook cell 2 (head)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `af606d09`

- [ ] **Step 1: Replace cell `af606d09` source**

Use NotebookEdit with `cell_id=af606d09`, `edit_mode=replace`, `cell_type=code`, source:

```python
# TASK 2 — Look at the first few rows of your dataset
#
# Now that `df` is loaded, you want to see what's actually in it:
# what are the column names? what type of values do they hold?
# (numbers, text, dates?)
#
# Your job:
#   - Display the first 5 rows of `df` so you and your team can read
#     the column names and get a feel for the data.
#
# Check the cheatsheet under "Inspecting" for the right function.
```

- [ ] **Step 2: Verify cell 2 no longer contains `.head()` call**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][1]['source']); assert 'df.head()' not in src; print('ok')"`
Expected output: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip code from notebook cell 2 (head)"
```

---

### Task 3: Strip notebook cell 3 (missing values)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `3871cee3`

- [ ] **Step 1: Replace cell `3871cee3` source**

Use NotebookEdit with `cell_id=3871cee3`, `edit_mode=replace`, `cell_type=code`, source:

```python
# TASK 3 — Find and handle missing values
#
# Real datasets often have gaps — empty cells, NaN, missing entries.
# Before you do any analysis, you need to know whether your dataset
# has any of those, and decide what to do about them.
#
# Your job:
#   1. Count how many missing values are in each column of `df`.
#   2. Look at the counts. If any column has missing values, decide
#      with your team how to handle them — common choices are:
#         - drop the rows that have missing values, or
#         - fill the missing cells with a sensible default
#           (e.g. 0, the column's mean, or the column's median).
#   3. Write the cleaning code so `df` no longer has missing values.
#
# Check the cheatsheet under "Cleaning" for the functions you need.
```

- [ ] **Step 2: Verify cell 3 no longer contains `print(`**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][2]['source']); assert 'print(' not in src and 'isnull' not in src; print('ok')"`
Expected output: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip code from notebook cell 3 (missing values)"
```

---

### Task 4: Strip notebook cell 4 (describe)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `564f921a`

- [ ] **Step 1: Replace cell `564f921a` source**

Use NotebookEdit with `cell_id=564f921a`, `edit_mode=replace`, `cell_type=code`, source:

```python
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
```

- [ ] **Step 2: Verify cell 4 no longer contains `.describe()`**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][3]['source']); assert '.describe()' not in src and 'print(' not in src; print('ok')"`
Expected output: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip code from notebook cell 4 (describe)"
```

---

### Task 5: Strip notebook cell 5 (sort/filter)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `2bc1092b`

- [ ] **Step 1: Replace cell `2bc1092b` source**

Use NotebookEdit with `cell_id=2bc1092b`, `edit_mode=replace`, `cell_type=code`, source:

```python
# TASK 5 — Sort and filter
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
```

- [ ] **Step 2: Verify cell 5 no longer contains `sort_values` call or example filter line**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][4]['source']); assert 'sort_values(' not in src and 'sorted_data' not in src; print('ok')"`
Expected output: `ok`

- [ ] **Step 3: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip code from notebook cell 5 (sort/filter)"
```

---

### Task 6: Strip notebook cell 6 (plot)

**Files:**
- Modify: `content/notebook.ipynb` — cell id `a1e4b985`

- [ ] **Step 1: Replace cell `a1e4b985` source**

Use NotebookEdit with `cell_id=a1e4b985`, `edit_mode=replace`, `cell_type=code`, source:

```python
# TASK 6 — Plot a relationship between two columns
#
# Pictures often reveal patterns that pure numbers hide. In this
# task you'll make a single plot that shows how two of your columns
# relate to each other.
#
# Your job:
#   1. Pick two columns from your dataset that you think might be
#      related, and decide which one is the X axis and which is Y.
#   2. Pick the type of plot that fits your data:
#         - sns.barplot     for one categorical column vs. one numeric
#         - sns.lineplot    for an ordered/time-like X axis
#         - sns.scatterplot for two numeric columns
#   3. Set a figure size so the plot is readable.
#   4. Add a title that describes what you're showing.
#   5. If your X labels overlap, rotate them.
#   6. Display the plot.
#
# Check the cheatsheet under "Plotting" for the functions you need.
```

- [ ] **Step 2: Verify cell 6 no longer contains seaborn/pyplot calls**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); src=''.join(nb['cells'][5]['source']); assert 'sns.' not in src and 'plt.' not in src; print('ok')"`
Expected output: `ok`

- [ ] **Step 3: Verify cell 7 (markdown report) is unchanged and notebook still has 7 cells**

Run: `python -c "import json; nb=json.load(open('content/notebook.ipynb')); assert len(nb['cells'])==7; assert 'Team Detective Report' in ''.join(nb['cells'][6]['source']); print('ok')"`
Expected output: `ok`

- [ ] **Step 4: Commit**

```bash
git add content/notebook.ipynb
git commit -m "strip code from notebook cell 6 (plot)"
```

---

### Task 7: Create student cheatsheet

**Files:**
- Create: `content/student_cheatsheet.md`

- [ ] **Step 1: Write the cheatsheet**

Create `content/student_cheatsheet.md` with exactly this content:

```markdown
# Student Cheatsheet

A quick reference for the functions you'll need in the workshop notebook. Each line shows the function and what it does. For full syntax details, ask Python directly with `help(function_name)` or check the official docs.

## Loading data

- `pd.read_csv("filename.csv")` — read a CSV file from disk and return it as a DataFrame.

## Inspecting

- `df.head()` — show the first 5 rows of the DataFrame.
- `df.head(n)` — show the first `n` rows.
- `df.columns` — list the column names.
- `df.shape` — tuple of `(number_of_rows, number_of_columns)`.
- `df.dtypes` — show the data type of each column.

## Cleaning

- `df.isnull().sum()` — count missing values per column.
- `df.fillna(value)` — replace missing values with `value` (e.g. `0`, a number, or `df["col"].mean()`).
- `df.dropna()` — drop any row that has at least one missing value.

## Analyzing

- `df.describe()` — table of count, mean, std, min, quartiles, and max for every numeric column.
- `df.sort_values(by="col_name", ascending=False)` — sort the DataFrame by a column, highest first.
- `df[df["col_name"] > value]` — keep only the rows where `col_name` is greater than `value`.
- `df["col_name"].mean()` — average of a single column.

## Plotting

- `plt.figure(figsize=(width, height))` — start a new figure with a given size in inches.
- `sns.barplot(data=df, x="col_x", y="col_y")` — bar chart, good for one categorical and one numeric column.
- `sns.lineplot(data=df, x="col_x", y="col_y")` — line chart, good for an ordered or time-like X axis.
- `sns.scatterplot(data=df, x="col_x", y="col_y")` — scatter plot, good for two numeric columns.
- `plt.title("My title")` — add a title to the current plot.
- `plt.xticks(rotation=45)` — rotate X-axis labels (use when they overlap).
- `plt.show()` — actually display the plot.
```

- [ ] **Step 2: Verify file content**

Run: `wc -l content/student_cheatsheet.md && grep -c '^## ' content/student_cheatsheet.md`
Expected: file has more than 20 lines; second number is `5` (five top-level `##` sections: Loading, Inspecting, Cleaning, Analyzing, Plotting).

- [ ] **Step 3: Commit**

```bash
git add content/student_cheatsheet.md
git commit -m "add student cheatsheet for workshop notebook"
```

---

### Task 8: Create TA cheatsheet

**Files:**
- Create: `content/ta_cheatsheet.md`

- [ ] **Step 1: Write the cheatsheet**

Create `content/ta_cheatsheet.md` with exactly this content:

````markdown
# TA Cheatsheet

A guide for TAs walking the room during the workshop. Each section maps to one task in `notebook.ipynb`.

**Important:** every team is working on a different `secret_dataset_<n>.csv`, so column names and value ranges will vary. The "Answer" code below uses placeholder names like `"some_column"` — students will need to substitute real column names from their own dataset.

When a team gets stuck, prefer the questions under "Hints to give" before showing them the code under "Answer".

---

## Task 1 — Load the dataset

### Answer

```python
df = pd.read_csv("secret_dataset_3.csv")  # replace 3 with the team's number
```

### Common pitfalls

- Wrong filename — typos, missing `.csv`, or referencing a file that isn't in `content/`.
- Missing the quotes around the filename, leading to a `NameError`.
- Forgetting to assign to `df`, so later cells fail with `NameError: name 'df' is not defined`.

### Hints to give

- "What's the exact name of your team's file in the file browser?"
- "How does pandas read a CSV file? Look at the cheatsheet under 'Loading data'."
- "What variable name do the next cells use? You'll need to assign your loaded data to that name."

---

## Task 2 — First few rows

### Answer

```python
df.head()
```

### Common pitfalls

- Calling `df.head` instead of `df.head()` — returns the method object, not the rows.
- Trying to print column names from memory instead of looking at the actual output.

### Hints to give

- "What method gives you the first few rows of a DataFrame?"
- "Look at the output carefully — what columns do you see, and what kinds of values are in them?"

---

## Task 3 — Missing values

### Answer

```python
# Count missing values
df.isnull().sum()

# Then either drop or fill, depending on what they see:
df = df.dropna()
# or, for a numeric column:
df = df.fillna(df.mean(numeric_only=True))
```

### Common pitfalls

- Calling `df.isnull()` without `.sum()` and getting a giant boolean DataFrame.
- Calling `df.dropna()` or `df.fillna(...)` without reassigning to `df`, so the cleaning has no effect.
- Filling with `0` when the column is text, or filling text columns with the mean.

### Hints to give

- "How would you count, per column, how many cells are missing?"
- "Did you actually save the cleaned DataFrame back into `df`?"
- "If you fill missing values with `0`, does that make sense for every column? What about the text ones?"

---

## Task 4 — Descriptive statistics

### Answer

```python
df.describe()
```

### Common pitfalls

- Expecting `describe()` to summarize text columns (by default it only shows numerics).
- Skimming past the output without reading the std and noticing what it says about the spread of values.

### Hints to give

- "There's one method that gives you mean, std, min and max for every numeric column at once. What is it?"
- "Look at the std for each column. Which columns are tightly clustered, and which are all over the place?"

---

## Task 5 — Sort and filter

### Answer

```python
# Sub-task A — Top 5
df.sort_values(by="some_column", ascending=False).head()

# Sub-task B — Filter above the mean
df[df["some_column"] > df["some_column"].mean()]
```

### Common pitfalls

- Using a column name with the wrong case or a typo — pandas raises `KeyError`.
- Forgetting `ascending=False`, so they get the bottom 5 instead of the top 5.
- Writing `df["col"] > df.mean()` (whole DataFrame) instead of `df["col"].mean()`.
- Not capturing the filtered result into a variable when they want to use it later.

### Hints to give

- "How exactly is your column spelled in `df.columns`? Capital letters and spaces matter."
- "By default, does `sort_values` sort smallest-to-largest or largest-to-smallest?"
- "When you say 'greater than the average', what are you taking the average of — the whole DataFrame or just one column?"

---

## Task 6 — Plot

### Answer

```python
plt.figure(figsize=(10, 6))
sns.barplot(data=df.head(15), x="x_column", y="y_column")
plt.title("My descriptive title")
plt.xticks(rotation=45)
plt.show()
```

### Common pitfalls

- Forgetting `plt.show()` — the plot may or may not appear depending on the environment.
- Choosing a barplot for two numeric columns (scatterplot would be the right call) or a scatterplot when one axis is categorical.
- Plotting all rows of a large dataset and getting an unreadable wall of bars.
- Long X-axis labels overlapping because they didn't rotate them.

### Hints to give

- "Are both of your axes numeric, or is one of them a category? That decides which plot type makes sense."
- "How many distinct values are on your X axis? If it's hundreds, what could you do to keep the plot readable?"
- "If your X labels are overlapping, what could you change?"
````

- [ ] **Step 2: Verify file content**

Run: `grep -c '^## Task ' content/ta_cheatsheet.md && grep -c '^### Answer' content/ta_cheatsheet.md && grep -c '^### Common pitfalls' content/ta_cheatsheet.md && grep -c '^### Hints to give' content/ta_cheatsheet.md`
Expected output: four lines, each `6` (six tasks, each with three subsections).

- [ ] **Step 3: Commit**

```bash
git add content/ta_cheatsheet.md
git commit -m "add TA cheatsheet with answers, pitfalls, and hints"
```

---

### Task 9: Final cross-check

**Files:** none modified — verification only.

- [ ] **Step 1: Verify all three deliverables exist and the notebook still parses**

Run:
```bash
python -c "
import json, os
nb = json.load(open('content/notebook.ipynb'))
assert len(nb['cells']) == 7, f'expected 7 cells, got {len(nb[\"cells\"])}'
assert os.path.exists('content/student_cheatsheet.md')
assert os.path.exists('content/ta_cheatsheet.md')
print('all deliverables present, notebook has 7 cells')
"
```
Expected output: `all deliverables present, notebook has 7 cells`

- [ ] **Step 2: Confirm no leftover Python in cells 2–6**

Run:
```bash
python -c "
import json
nb = json.load(open('content/notebook.ipynb'))
forbidden = ['df.head()', 'df.isnull', 'df.describe()', 'sort_values(', 'sns.', 'plt.figure', 'plt.show']
for i in range(1, 6):
    src = ''.join(nb['cells'][i]['source'])
    for f in forbidden:
        assert f not in src, f'cell {i+1} still contains {f!r}'
print('cells 2-6 are clean')
"
```
Expected output: `cells 2-6 are clean`

- [ ] **Step 3: No commit needed for verification-only task.**
