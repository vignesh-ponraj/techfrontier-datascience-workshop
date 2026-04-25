# Dataset Fit and Task Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix the three datasets that don't load cleanly, expand the workshop notebook from 6 to 9 tasks so every dataset shape works and the workshop fills 2 hours, and update both cheatsheets to match.

**Architecture:** Three independent groups of changes — (1) rewrite three CSVs, (2) restructure `notebook.ipynb` (insert 3 new task cells, renumber 2 existing cells), (3) edit the two cheatsheet markdown files. Each task in the plan produces one self-contained commit.

**Tech Stack:** Python 3 (`csv` stdlib only — no pandas required for the rewrites), Jupyter notebook (ipynb JSON, edited via `NotebookEdit`), Markdown.

---

## Spec

Source spec: [2026-04-25-dataset-fit-and-task-expansion-design.md](../specs/2026-04-25-dataset-fit-and-task-expansion-design.md)

Refer to the spec for rationale, dataset audit details, and the explicit list of preserved/discarded columns. This plan implements the spec exactly.

---

## File Structure

| Path | Action | Net effect |
| --- | --- | --- |
| `content/secret_dataset_2.csv` | rewrite | utf-8, 16-column subset, single header row |
| `content/secret_dataset_11.csv` | rewrite | `**` → empty cell |
| `content/secret_dataset_12.csv` | rewrite | 15-column subset |
| `content/notebook.ipynb` | modify | 3 new task cells inserted, 2 existing cells get heading renumbered |
| `content/student_cheatsheet.md` | modify | one new "Building" section, value_counts and groupby entries |
| `content/ta_cheatsheet.md` | modify | renumber existing sections, 3 new per-task sections, 15 per-dataset sections |

All other files (`README.md`, `requirements.txt`, `data.csv`, the spec/plan docs) are untouched.

---

### Task 1: Preprocess `secret_dataset_11.csv` (Librarians) — replace `**` with empty cell

**Files:**
- Modify: `content/secret_dataset_11.csv`

**Background:** every numeric cell is `**` (BLS confidentiality suppression code). Pandas reads these as text, so all four numeric columns become `object` and the team can't run `describe`/`sort`/etc. Audit shows only 2 of 373 rows are fully suppressed, so a simple `**` → empty replacement preserves nearly all rows.

- [ ] **Step 1: Write a one-shot rewrite script and execute it**

Run from the repo root:
```bash
python3 - <<'PY'
import csv
src = 'content/secret_dataset_11.csv'
with open(src, newline='') as f:
    rows = list(csv.reader(f))
# Replace literal '**' cells with empty string (which pandas reads as NaN)
new_rows = [[('' if cell == '**' else cell) for cell in row] for row in rows]
with open(src, 'w', newline='') as f:
    csv.writer(f).writerows(new_rows)
print(f"rewrote {src}: {len(new_rows)-1} data rows")
PY
```
Expected output: `rewrote content/secret_dataset_11.csv: 373 data rows`

- [ ] **Step 2: Verify no `**` cells remain**

```bash
python3 -c "
import csv
with open('content/secret_dataset_11.csv') as f:
    rows = list(csv.reader(f))
bad = sum(1 for r in rows for c in r if c == '**')
print(f'remaining ** cells: {bad}')
assert bad == 0
print('ok')
"
```
Expected output: `remaining ** cells: 0` followed by `ok`.

- [ ] **Step 3: Verify the four numeric columns parse as float**

```bash
python3 -c "
import csv
with open('content/secret_dataset_11.csv') as f:
    rows = list(csv.reader(f))
hdr = rows[0]; data = rows[1:]
expected_numeric = ['tot_emp', 'emp_prse', 'jobs_1000', 'loc_quotient']
for col in expected_numeric:
    ci = hdr.index(col)
    bad = []
    for r in data:
        v = r[ci]
        if v == '': continue
        try: float(v)
        except: bad.append(v)
    assert not bad, f'{col!r}: non-numeric values found: {bad[:3]}'
    print(f'{col!r}: ok')
"
```
Expected: four `'colname': ok` lines, no errors.

- [ ] **Step 4: Commit**

```bash
git add content/secret_dataset_11.csv
git commit -m "preprocess librarians dataset: replace BLS ** suppression with empty cells"
```

---

### Task 2: Preprocess `secret_dataset_12.csv` (Nutrition) — subset to 15 columns

**Files:**
- Modify: `content/secret_dataset_12.csv`

**Background:** the original has 1093 columns — `df.head()` is unreadable for entry-level students. We subset to a 15-column slice that gives both binary categoricals (good for `value_counts`) and food frequency/quantity numerics (good for `describe`/`groupby`).

- [ ] **Step 1: Write a one-shot subset script and execute it**

```bash
python3 - <<'PY'
import csv
src = 'content/secret_dataset_12.csv'
keep = [
    'ID', 'cancer', 'diabetes', 'heart_disease', 'belly',
    'ever_smoked', 'currently_smoke',
    'cat', 'dog',
    'EGGSFREQ', 'EGGSQUAN',
    'COLDCEREALFREQ', 'COLDCEREALQUAN',
    'WHITERICEFREQ', 'WHITERICEQUAN',
]
with open(src, newline='') as f:
    rows = list(csv.reader(f))
hdr = rows[0]
# Headers are quoted in the file; csv.reader strips quotes, so use literal names.
missing = [c for c in keep if c not in hdr]
assert not missing, f'columns not found in source: {missing}'
indices = [hdr.index(c) for c in keep]
new_rows = [keep] + [[r[i] for i in indices] for r in rows[1:]]
with open(src, 'w', newline='') as f:
    csv.writer(f).writerows(new_rows)
print(f'rewrote {src}: {len(new_rows)-1} data rows, {len(keep)} columns')
PY
```
Expected output: `rewrote content/secret_dataset_12.csv: 54 data rows, 15 columns`

- [ ] **Step 2: Verify column count and names**

```bash
python3 -c "
import csv
with open('content/secret_dataset_12.csv') as f:
    rows = list(csv.reader(f))
hdr = rows[0]
expected = ['ID','cancer','diabetes','heart_disease','belly','ever_smoked','currently_smoke','cat','dog','EGGSFREQ','EGGSQUAN','COLDCEREALFREQ','COLDCEREALQUAN','WHITERICEFREQ','WHITERICEQUAN']
assert hdr == expected, f'header mismatch: {hdr}'
print(f'ok, {len(rows)-1} rows, {len(hdr)} cols')
"
```
Expected output: `ok, 54 rows, 15 cols`.

- [ ] **Step 3: Commit**

```bash
git add content/secret_dataset_12.csv
git commit -m "preprocess nutrition dataset: subset to 15 columns"
```

---

### Task 3: Preprocess `secret_dataset_2.csv` (Star Wars) — utf-8 + 16-column subset + flat header

**Files:**
- Modify: `content/secret_dataset_2.csv`

**Background:** the original is latin-1 encoded (default `pd.read_csv` raises `UnicodeDecodeError`) and has a two-row header (sub-header in row 1 of data). We rewrite as utf-8 with a single flat header and 16 readable columns. The 6 rank columns become numeric; the rest stay text.

Source-column indices were verified during planning (0-based). The mapping:

| Source index | New column name |
| --- | --- |
| 0 | `RespondentID` |
| 1 | `has_seen_franchise` |
| 2 | `is_fan_starwars` |
| 9 | `rank_ep1_phantom_menace` |
| 10 | `rank_ep2_clones` |
| 11 | `rank_ep3_sith` |
| 12 | `rank_ep4_new_hope` |
| 13 | `rank_ep5_empire` |
| 14 | `rank_ep6_jedi` |
| 29 | `shot_first` |
| 32 | `is_fan_startrek` |
| 33 | `gender` |
| 34 | `age` |
| 35 | `household_income` |
| 36 | `education` |
| 37 | `region` |

- [ ] **Step 1: Write a one-shot rewrite script and execute it**

```bash
python3 - <<'PY'
import csv
src = 'content/secret_dataset_2.csv'
indices = [0, 1, 2, 9, 10, 11, 12, 13, 14, 29, 32, 33, 34, 35, 36, 37]
new_header = [
    'RespondentID',
    'has_seen_franchise', 'is_fan_starwars',
    'rank_ep1_phantom_menace', 'rank_ep2_clones', 'rank_ep3_sith',
    'rank_ep4_new_hope', 'rank_ep5_empire', 'rank_ep6_jedi',
    'shot_first', 'is_fan_startrek',
    'gender', 'age', 'household_income', 'education', 'region',
]
assert len(indices) == len(new_header) == 16

with open(src, encoding='latin-1', newline='') as f:
    rows = list(csv.reader(f))

# Drop original header rows (row 0 = section headers, row 1 = sub-headers).
# Data starts at row 2.
data_in = rows[2:]
data_out = [[r[i] for i in indices] for r in data_in]
new_rows = [new_header] + data_out

with open(src, encoding='utf-8', mode='w', newline='') as f:
    csv.writer(f).writerows(new_rows)
print(f'rewrote {src} as utf-8: {len(data_out)} data rows, {len(new_header)} columns')
PY
```
Expected output: `rewrote content/secret_dataset_2.csv as utf-8: 1186 data rows, 16 columns`

- [ ] **Step 2: Verify utf-8 loads cleanly and rank columns parse as int**

```bash
python3 -c "
import csv
with open('content/secret_dataset_2.csv', encoding='utf-8') as f:
    rows = list(csv.reader(f))
hdr = rows[0]
expected = ['RespondentID','has_seen_franchise','is_fan_starwars','rank_ep1_phantom_menace','rank_ep2_clones','rank_ep3_sith','rank_ep4_new_hope','rank_ep5_empire','rank_ep6_jedi','shot_first','is_fan_startrek','gender','age','household_income','education','region']
assert hdr == expected, f'header mismatch: {hdr}'
data = rows[1:]
print(f'rows: {len(data)}, cols: {len(hdr)}')
# rank columns must be empty or integers 1-6
for col in ['rank_ep1_phantom_menace','rank_ep2_clones','rank_ep3_sith','rank_ep4_new_hope','rank_ep5_empire','rank_ep6_jedi']:
    ci = hdr.index(col)
    bad = []
    for r in data:
        v = r[ci]
        if v == '': continue
        try:
            iv = int(v)
            if iv < 1 or iv > 6: bad.append(v)
        except: bad.append(v)
    assert not bad, f'{col!r}: bad values {bad[:3]}'
print('ok')
"
```
Expected output: a row count line, then `ok`.

- [ ] **Step 3: Commit**

```bash
git add content/secret_dataset_2.csv
git commit -m "preprocess star wars dataset: utf-8 + 16-column subset + flat header"
```

---

### Task 4: Insert three new task cells into `notebook.ipynb`

**Files:**
- Modify: `content/notebook.ipynb`

**Background:** the new task sequence has three new cells: Task 5 (`value_counts`), Task 7 (derive a column), Task 8 (`groupby` mean). They go between existing cells. Insertion order matters because each `NotebookEdit` insert anchors on a cell ID that must already exist.

**Insert in the order below.** The trick: insert the *latest* new cell first (after the sort/filter cell `2bc1092b`). Then insert the next-latest at the same anchor — it'll land between the sort/filter cell and the cell we just inserted. Repeat. This way every insert anchors on a known, pre-existing cell ID.

After all three inserts, the cell order will be: load, head, missing, describe, **value_counts**, sort/filter, **derive**, **groupby**, plot, report — total 10 cells.

- [ ] **Step 1: Insert new groupby cell (becomes Task 8) after the sort/filter cell**

Use the `NotebookEdit` tool with:
- `notebook_path`: `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/notebook.ipynb`
- `edit_mode`: `insert`
- `cell_id`: `2bc1092b` (insert *after* this cell)
- `cell_type`: `code`
- `new_source`:

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
```

- [ ] **Step 2: Insert new derive-column cell (becomes Task 7) after the sort/filter cell**

Use `NotebookEdit` with the same `notebook_path`, `edit_mode=insert`, `cell_id=2bc1092b`, `cell_type=code`, and `new_source`:

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
#   - Confirm it worked by running `df.head()` again — your new column
#     should appear at the right of the table.
#
# Check the cheatsheet under "Building" for the syntax pattern.
```

- [ ] **Step 3: Insert new value_counts cell (becomes Task 5) after the describe cell**

Use `NotebookEdit` with `notebook_path` as above, `edit_mode=insert`, `cell_id=564f921a`, `cell_type=code`, `new_source`:

```
# TASK 5 — Count categories
#
# `describe()` summarized your numeric columns. Now do the parallel
# thing for categorical columns — columns whose values are categories,
# words, or labels rather than numbers.
#
# Your job:
#   - Pick a categorical column from `df` (look at the columns from
#     Task 2).
#   - Count how many times each unique value appears in that column.
#
# Then look at the result and discuss with your team:
#   - Which values are most common?
#   - Which are rare?
#   - Are there any surprises?
#
# Check the cheatsheet under "Inspecting" for the function you need.
```

- [ ] **Step 4: Verify the notebook has 10 cells in the correct order**

Run from the repo root:
```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
assert len(nb['cells']) == 10, f'expected 10 cells, got {len(nb[\"cells\"])}'
heads = [''.join(c['source']).splitlines()[0] if c['source'] else '' for c in nb['cells']]
expected_headers = [
    'import pandas as pd',                              # cell 0 = Task 1 (Load)
    '# TASK 2 — Look at the first few rows of your dataset',
    '# TASK 3 — Find and handle missing values',
    '# TASK 4 — Generate a statistical summary',
    '# TASK 5 — Count categories',
    '# TASK 5 — Sort and filter',                       # to be renumbered in Task 5 of plan
    '# TASK 7 — Derive a new column',
    '# TASK 8 — Group and average',
    '# TASK 6 — Plot a relationship between two columns',# to be renumbered in Task 5 of plan
    '### Team Detective Report',
]
for i, (got, want) in enumerate(zip(heads, expected_headers)):
    assert got == want, f'cell {i}: got {got!r}, expected {want!r}'
print('ok, 10 cells in correct order; sort/filter and plot still need renumber')
"
```
Expected output: `ok, 10 cells in correct order; sort/filter and plot still need renumber`

- [ ] **Step 5: Commit**

```bash
git add content/notebook.ipynb
git commit -m "add three new task cells to notebook (value_counts, derive, groupby)"
```

---

### Task 5: Renumber two existing notebook task headings

**Files:**
- Modify: `content/notebook.ipynb`

**Background:** after Task 4, two existing cells still display the old TASK numbers. We update only the heading line in each.

- [ ] **Step 1: Update sort/filter cell (`2bc1092b`) heading from `TASK 5` to `TASK 6`**

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
```

- [ ] **Step 2: Update plot cell (`a1e4b985`) heading from `TASK 6` to `TASK 9`**

Use `NotebookEdit` with `cell_id=a1e4b985`, `edit_mode=replace`, `cell_type=code`, `new_source`:

```
# TASK 9 — Plot a relationship between two columns
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

- [ ] **Step 3: Verify the cell sequence**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
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
    assert g == w, f'cell {i}: got {g!r}, expected {w!r}'
print('ok, all 10 cell headings are correct')
"
```
Expected output: `ok, all 10 cell headings are correct`

- [ ] **Step 4: Commit**

```bash
git add content/notebook.ipynb
git commit -m "renumber sort/filter to task 6 and plot to task 9 in notebook"
```

---

### Task 6: Update student cheatsheet — add `value_counts`, "Building" section, and `groupby`

**Files:**
- Modify: `content/student_cheatsheet.md`

**Background:** the new tasks reference cheatsheet sections `Inspecting`, `Building`, and `Analyzing`. We need to add one entry to Inspecting, create a new Building section between Cleaning and Analyzing, and add one entry to Analyzing.

- [ ] **Step 1: Replace the entire file with the updated content**

Use the `Write` tool with file path `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/student_cheatsheet.md` and content:

```
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
- `df["col_name"].value_counts()` — count how many times each unique value appears in a column.

## Cleaning

- `df.isnull().sum()` — count missing values per column.
- `df.fillna(value)` — replace missing values with `value` (e.g. `0`, a number, or `df["col"].mean()`).
- `df.dropna()` — drop any row that has at least one missing value.

## Building

- `df["new_col"] = df["a"] + df["b"]` — create a new column that's the sum of two existing columns.
- `df["new_col"] = df["a"] - df["b"]` — create a new column that's a difference.
- `df["new_col"] = df["a"] / df["b"]` — create a new column that's a ratio.

## Analyzing

- `df.describe()` — table of count, mean, std, min, quartiles, and max for every numeric column.
- `df.sort_values(by="col_name", ascending=False)` — sort the DataFrame by a column, highest first.
- `df[df["col_name"] > value]` — keep only the rows where `col_name` is greater than `value`.
- `df["col_name"].mean()` — average of a single column.
- `df.groupby("cat_col")["num_col"].mean()` — average of `num_col` for each category in `cat_col`.

## Plotting

- `plt.figure(figsize=(width, height))` — start a new figure with a given size in inches.
- `sns.barplot(data=df, x="col_x", y="col_y")` — bar chart, good for one categorical and one numeric column.
- `sns.lineplot(data=df, x="col_x", y="col_y")` — line chart, good for an ordered or time-like X axis.
- `sns.scatterplot(data=df, x="col_x", y="col_y")` — scatter plot, good for two numeric columns.
- `plt.title("My title")` — add a title to the current plot.
- `plt.xticks(rotation=45)` — rotate X-axis labels (use when they overlap).
- `plt.show()` — actually display the plot.
```

- [ ] **Step 2: Verify section order and presence of new entries**

```bash
grep -E '^## ' content/student_cheatsheet.md
grep -c 'value_counts' content/student_cheatsheet.md
grep -c 'groupby' content/student_cheatsheet.md
```
Expected:
- First command lists six sections in order: `## Loading data`, `## Inspecting`, `## Cleaning`, `## Building`, `## Analyzing`, `## Plotting`.
- Second command outputs `1`.
- Third command outputs `1`.

- [ ] **Step 3: Commit**

```bash
git add content/student_cheatsheet.md
git commit -m "update student cheatsheet: add value_counts, Building section, and groupby"
```

---

### Task 7: Update TA cheatsheet — Part 1 (per-task half: renumber + 3 new sections + preamble)

**Files:**
- Modify: `content/ta_cheatsheet.md`

**Background:** restructure the file's per-task half. Renumber Task 5 (sort/filter) to Task 6, Task 6 (plot) to Task 9. Insert three new per-task sections: Task 5 (Count categories), Task 7 (Derive a new column), Task 8 (Group and average). Update the preamble to mention the upcoming Datasets half.

This task overwrites the entire file because the Datasets half doesn't exist yet — Task 8 will append it.

- [ ] **Step 1: Replace the entire file with the updated per-task content**

Use the `Write` tool with file path `/Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/ta_cheatsheet.md` and content:

````
# TA Cheatsheet

A guide for TAs walking the room during the workshop. The file has two halves:

- **Per-task guidance** (below) — for each of the 9 tasks: the answer code, common pitfalls, and Socratic hints. Use this when a team is stuck on *what to do*.
- **Datasets** (further down) — one entry per `secret_dataset_<n>.csv` with suggested column choices and dataset-specific quirks. Use this when a team is stuck on *what their data is*.

**Important:** every team is working on a different `secret_dataset_<n>.csv`, so column names and value ranges will vary. The "Answer" code below uses placeholder names like `"some_column"` — students will need to substitute real column names from their own dataset (the per-dataset section suggests good picks).

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

## Task 5 — Count categories

### Answer

```python
df["some_categorical_column"].value_counts()
```

### Common pitfalls

- Picking a numeric column with hundreds of unique values — output is unreadable. Steer them to a column with a small number of distinct values (e.g. gender, region, yes/no fields).
- Calling `value_counts()` on the whole DataFrame instead of one column — `df.value_counts()` works in modern pandas but produces a multi-row index that confuses beginners.
- Treating the output as a DataFrame — it's a Series, indexed by the unique values.

### Hints to give

- "Which of your columns hold *labels* rather than numbers? Pick one of those."
- "How would you count how often each unique value appears? Look at the cheatsheet under 'Inspecting'."
- "Out of the values that came back, which is most common? Which is rare?"

---

## Task 6 — Sort and filter

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

## Task 7 — Derive a new column

### Answer

```python
# Example shapes:
df["total"] = df["a"] + df["b"]
df["diff"]  = df["a"] - df["b"]
df["ratio"] = df["a"] / df["b"]
```

### Common pitfalls

- Trying to add or divide text columns — pandas concatenates strings or raises a `TypeError`. The columns being combined must be numeric (or both text for concat).
- Forgetting the assignment back to a column: writing `df["a"] + df["b"]` alone produces a Series but doesn't store it.
- Naming the new column the same as an existing column and silently overwriting it.
- Dividing by a column that contains zeros — they get `inf` or `NaN` in some rows.

### Hints to give

- "What's a question about your data that needs a column you don't have yet?"
- "Which two columns could you combine — by adding, subtracting, or dividing — to answer that question?"
- "Once you've created the new column, how do you check it actually appeared in `df`?"

---

## Task 8 — Group and average

### Answer

```python
df.groupby("some_categorical_column")["some_numeric_column"].mean()
```

### Common pitfalls

- Swapping the categorical and numeric columns — `df.groupby("num")["cat"].mean()` raises `TypeError` because you can't average text.
- Picking a categorical column with hundreds of unique values — the output is one row per category, so it's unreadable.
- Forgetting to look at *both* the highest and lowest groups in the result.
- Using `.mean()` on a column that's mostly NaN — the means are fine, but the team may not realize how few rows back each group.

### Hints to give

- "Which column would you use to split your data into groups? It should be a category column with not too many distinct values."
- "Which numeric column would you average inside each group?"
- "Which group came out on top? Was that surprising? What about the lowest one?"

---

## Task 9 — Plot

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

(Note: the file ends after the Task 9 block. Task 8 of this plan adds the Datasets half on top of this.)

- [ ] **Step 2: Verify per-task structure**

```bash
grep -c '^## Task ' content/ta_cheatsheet.md
grep -c '^### Answer' content/ta_cheatsheet.md
grep -c '^### Common pitfalls' content/ta_cheatsheet.md
grep -c '^### Hints to give' content/ta_cheatsheet.md
grep -E '^## Task ' content/ta_cheatsheet.md
```
Expected:
- First four counts: each `9`.
- Last command lists tasks in order: Task 1, Task 2, Task 3, Task 4, Task 5 (Count categories), Task 6 (Sort and filter), Task 7 (Derive a new column), Task 8 (Group and average), Task 9 (Plot).

- [ ] **Step 3: Commit**

```bash
git add content/ta_cheatsheet.md
git commit -m "update TA cheatsheet per-task half: 3 new sections, renumbering, preamble"
```

---

### Task 8: Append the per-dataset half to the TA cheatsheet

**Files:**
- Modify: `content/ta_cheatsheet.md`

**Background:** add a new `## Datasets` heading at the bottom of the file, followed by 15 subsections (`### Dataset 1` through `### Dataset 15`). Each entry has four labeled parts: **Source**, **Shape**, **Suggested columns**, **Quirks**. Per-dataset content has been written based on the audit and post-preprocessing state.

The "Suggested columns" subsection lists picks for: numeric column for sort/filter and groupby's numeric side, categorical column for value_counts and groupby's grouping side, two columns for plot, and one or two columns suitable for the derived-column task.

- [ ] **Step 1: Append the Datasets half to the file**

Use the `Bash` tool with a heredoc append:

```bash
cat >> /Users/vigneshponraj/Documents/github/techfrontier-datascience-workshop/content/ta_cheatsheet.md <<'TA_DATASETS_EOF'

---

## Datasets

One entry per `secret_dataset_<n>.csv`. Use these to anticipate where a specific team is likely to get stuck and to suggest sensible column picks when they ask "which column should we use?"

---

### Dataset 1

**Source:** FiveThirtyEight Pulitzer circulation data (newspapers + Pulitzer prize counts).

**Shape:** 50 rows × 7 columns.

**Suggested columns:**
- numeric: `Pulitzer Prize Winners and Finalists, 1990-2014` (or either of the two narrower windows).
- categorical: `Newspaper` (50 unique values — fine for value_counts since each appears once, but `groupby` is meaningless on this column).
- plot: `Newspaper` (X) vs the `1990-2014` prize count (Y); use `df.head(15)` so labels are readable.
- derived column: `Pulitzer Prize Winners and Finalists, 1990-2003` + `2004-2014` summed (and verify it equals the `1990-2014` column — fun consistency check).

**Quirks:**
- `Daily Circulation, 2004` and `Daily Circulation, 2013` look numeric but pandas reads them as text (commas in the numbers). `Change in Daily Circulation, 2004-2013` is text too (`+13%` / `-24%`). The team has 3 real numerics — that's enough. Curious teams can clean the circulation columns with `df["col"].str.replace(",","").astype(int)`.
- No missing values, so Task 3 will be quick — they'll see all zeros in `isnull().sum()` and move on.
- `groupby` on `Newspaper` produces one row per group (each newspaper is unique). Steer them to think of this as a "small ranked list" dataset, not a grouping dataset.

---

### Dataset 2

**Source:** FiveThirtyEight Star Wars survey *(preprocessed: utf-8, 16 columns, single header row)*.

**Shape:** ~1186 rows × 16 columns.

**Suggested columns:**
- numeric: any of the six `rank_ep<N>_<title>` columns (values 1–6).
- categorical: `gender`, `education`, `region`, `is_fan_starwars`, `shot_first` — all small-cardinality.
- plot: `region` (X) vs the average rank of one episode (Y) — works well as a barplot of `groupby` output.
- derived column: `df["avg_prequel_rank"] = (df["rank_ep1_phantom_menace"] + df["rank_ep2_clones"] + df["rank_ep3_sith"]) / 3`.

**Quirks:**
- **Rank columns invert intuition:** `1` is the *favorite*, `6` is the *least favorite*. If they sort `ascending=False` they get the worst-ranked, not the best. Make them think about which direction they want.
- Many NaN in rank columns from people who didn't see all six films. Encourage `dropna(subset=["rank_ep1_phantom_menace"])` before averaging or sorting.
- `RespondentID` is numeric but meaningless to analyze — make sure they don't pick it.

---

### Dataset 3

**Source:** FiveThirtyEight Thanksgiving survey.

**Shape:** ~1058 rows × 65 columns.

**Suggested columns:**
- numeric: only one truly numeric column; this team will do most analysis on categoricals.
- categorical: `Do you celebrate Thanksgiving?`, `What is typically the main dish at your Thanksgiving dinner?`, `What kind of stuffing/dressing do you typically have?`, `US Region`. All have small cardinality.
- plot: a `value_counts` of the main dish or stuffing column lends itself to a barplot of counts.
- derived column: tricky on this dataset — accept any reasonable string concat (e.g. combining region + age into a single label) or a recode (e.g. `df["adult"] = df["Age"] != "18 - 29"`).

**Quirks:**
- Survey data with skip-logic — many columns are 50%+ missing. Don't try to `dropna()` over all columns, the DataFrame will go to zero rows. Recommend dropping NaN only for the specific columns being used in a step.
- `describe()` and `sort_values` will be thin. Lean hard into `value_counts` (Task 5) and `groupby` (Task 8) — those are where this dataset shines.
- For Task 8, suggest grouping by `US Region` and counting (e.g. `.size()` instead of `.mean()`) if the team can't find a numeric column to average. Document this as the alternative shape.

---

### Dataset 4

**Source:** FiveThirtyEight unisex names — names whose male/female usage is roughly balanced.

**Shape:** 919 rows × 6 columns.

**Suggested columns:**
- numeric: `total`, `male_share`, `female_share`, `gap`.
- categorical: `name` is technically categorical but each is unique; tell them this dataset has no good column for `value_counts` and they'll get most insight from sort/filter and plot.
- plot: `name` (X) vs `total` (Y) — but only after sorting and `head(15)`.
- derived column: `df["dominant"] = df["male_share"] - df["female_share"]` — a signed gap that sorts in a meaningful direction.

**Quirks:**
- This is a numeric-friendly dataset — most tasks "just work."
- `value_counts` is awkward — there are no real categorical columns. The TA can suggest binning a numeric column (`pd.cut(df["total"], 4).value_counts()`) as a stretch, but if it's confusing, accept that this team will skip the most categorical-flavored part of Task 5.

---

### Dataset 5

**Source:** FiveThirtyEight weather check survey.

**Shape:** 928 rows × 9 columns.

**Suggested columns:**
- numeric: only `RespondentID` is numeric — useless to analyze.
- categorical: `Do you typically check a daily weather report?`, `How do you typically check the weather?`, `A typical day...`, `Age`, `What is your gender?`, `How much total combined money...`, `US Region`. All small-cardinality.
- plot: barplot of one categorical's `value_counts`, or `groupby` and `.size()` instead of `.mean()`.
- derived column: same situation as Dataset 3 — accept a string concat or a recode.

**Quirks:**
- Same shape as Dataset 3. Treat similarly: `value_counts` and `groupby ... .size()` are the productive paths. `describe()` will return basically just stats on `RespondentID`.
- `groupby ... .mean()` doesn't fit. Suggest `df.groupby("US Region")["RespondentID"].count()` or simply `df.groupby("US Region").size()` so the team can still answer "which region has the most respondents."

---

### Dataset 6

**Source:** FiveThirtyEight new voter registrations.

**Shape:** 106 rows × 4 columns. Columns: `Jurisdiction`, `Year`, `Month`, `New registered voters`.

**Suggested columns:**
- numeric: `New registered voters`.
- categorical: `Jurisdiction` (small set of US states), `Year`, `Month`.
- plot: `Jurisdiction` (X) vs `groupby` average of `New registered voters` (Y).
- derived column: a string concat of period: `df["period"] = df["Year"].astype(str) + "-" + df["Month"].astype(str)`. (A pure numeric derivation is awkward here — the dataset is already in long format and the natural numeric is just the one count.)

**Quirks:**
- Long-format dataset (each row is a month-jurisdiction observation), so `groupby("Jurisdiction")["New registered voters"].sum()` answers "which state registered the most voters?" — this is more useful than `mean()` here. Tell the team `mean` works for Task 8 as written, but `sum` is more interpretable.
- `value_counts` on `Jurisdiction` returns "how many months were reported per state" — slightly meta, but fine.
- Small (106 rows) — they'll finish quickly, so save time for the Detective Report.

---

### Dataset 7

**Source:** FiveThirtyEight alcohol consumption — servings per person by country.

**Shape:** 193 rows × 5 columns.

**Suggested columns:**
- numeric: `beer_servings`, `spirit_servings`, `wine_servings`, `total_litres_of_pure_alcohol`.
- categorical: `country` is unique-per-row.
- plot: `country` (X) vs `total_litres_of_pure_alcohol` (Y) after `head(15)`.
- derived column: `df["beer_share"] = df["beer_servings"] / (df["beer_servings"] + df["spirit_servings"] + df["wine_servings"])`.

**Quirks:**
- A clean numeric-friendly dataset. Every task fits.
- `value_counts` on `country` returns 193 rows of `1` — not useful. The team should pick a derived bucket (e.g. binning `beer_servings`) for Task 5 if they want a meaningful count, or accept that `value_counts` on this dataset is mostly trivia.

---

### Dataset 8

**Source:** FiveThirtyEight candy power ranking.

**Shape:** 85 rows × 13 columns.

**Suggested columns:**
- numeric: `winpercent`, `sugarpercent`, `pricepercent`.
- categorical: any of the 0/1 binary columns (`chocolate`, `fruity`, `caramel`, `peanutyalmondy`, `nougat`, `crispedricewafer`, `hard`, `bar`, `pluribus`).
- plot: `competitorname` (X) vs `winpercent` (Y), sorted, `head(15)`.
- derived column: `df["sweetness_to_price"] = df["sugarpercent"] / df["pricepercent"]`.

**Quirks:**
- The 0/1 binary columns work as both categorical (for `value_counts`) and numeric (mean of a 0/1 column = the proportion). Excellent dataset for `groupby`: `df.groupby("chocolate")["winpercent"].mean()` gives a great "do chocolate candies win more?" answer.
- `describe()` shows the binary columns as numerics with mean ≈ proportion — students may notice this and ask why. Good teaching moment.

---

### Dataset 9

**Source:** FiveThirtyEight college majors — graduate students by major.

**Shape:** 173 rows × 22 columns.

**Suggested columns:**
- numeric: `Grad_total`, `Grad_employed`, `Grad_unemployment_rate`, `Grad_median`, `Nongrad_median`.
- categorical: `Major_category` (16 distinct values — perfect for `groupby`).
- plot: `Major_category` (X) vs `Grad_unemployment_rate` (Y) after `groupby` and sort.
- derived column: `df["employed_share"] = df["Grad_employed"] / df["Grad_total"]` — fraction of grads who are employed.

**Quirks:**
- `Major_category` is the dream column for `groupby` — small cardinality, meaningful groupings, lots of numerics to average.
- `Major` (the column with 173 unique major names) is too granular for `value_counts` — push them toward `Major_category` instead.
- This dataset already has a `Grad_premium` column (median pay difference vs non-grads). If a team derives that themselves, they can verify against the existing column — fun consistency check.

---

### Dataset 10

**Source:** FiveThirtyEight Fandango movie ratings (and other rating sources).

**Shape:** 510 rows × 4 columns.

**Suggested columns:**
- numeric: the rating columns (e.g. `STARS`, `RATING`, `VOTES` — verify exact names from `df.columns`).
- categorical: `FILM` is unique per row.
- plot: `FILM` (X) vs `STARS` (Y), sorted, `head(15)`.
- derived column: difference between two rating columns, e.g. `df["star_inflation"] = df["STARS"] - df["RATING"]`.

**Quirks:**
- 510 unique films — `value_counts` on FILM is trivia. Suggest that this dataset's strength is sorting and filtering.
- The "STARS" column reflects what was *displayed*; `RATING` is the underlying number. The difference between them is the famous Fandango inflation finding — a good story for the Detective Report.

---

### Dataset 11

**Source:** FiveThirtyEight librarians by Metropolitan Statistical Area *(preprocessed: BLS `**` codes replaced with empty cells so pandas reads them as NaN)*.

**Shape:** 373 rows × 6 columns.

**Suggested columns:**
- numeric: `tot_emp`, `emp_prse`, `jobs_1000`, `loc_quotient`.
- categorical: `prim_state`.
- plot: `prim_state` (X) vs the average `loc_quotient` per state (Y), via `groupby` then plot.
- derived column: `df["jobs_per_loc"] = df["jobs_1000"] / df["loc_quotient"]` — slightly contrived but legal.

**Quirks:**
- Only 2 of 373 rows have any suppressed cells, so Task 3 (missing values) is quick — `isnull().sum()` shows tiny counts. Document this expectation.
- `loc_quotient` (location quotient — relative concentration vs national average) is the most interesting numeric. Sort by it to find the MSAs where librarian work is most over- or under-represented.
- `area_name` is unique per row (each MSA is its own entry), so `value_counts` on it is trivia. Use `prim_state` for `value_counts` and `groupby`.

---

### Dataset 12

**Source:** FiveThirtyEight nutrition study *(preprocessed: subset to 15 columns)*.

**Shape:** 54 rows × 15 columns.

**Suggested columns:**
- numeric: `EGGSFREQ`, `EGGSQUAN`, `COLDCEREALFREQ`, `COLDCEREALQUAN`, `WHITERICEFREQ`, `WHITERICEQUAN`.
- categorical: any of the 0/1 columns: `cancer`, `diabetes`, `heart_disease`, `belly`, `ever_smoked`, `currently_smoke`, `cat`, `dog`.
- plot: `EGGSFREQ` (X) vs `EGGSQUAN` (Y) as a scatterplot, OR a barplot of `df.groupby("cat")["EGGSFREQ"].mean()` ("do cat owners eat more eggs?").
- derived column: `df["eggs_per_serving"] = df["EGGSQUAN"] / df["EGGSFREQ"]` — average serving size when they do eat eggs.

**Quirks:**
- The binary 0/1 columns serve double duty (categorical for `value_counts`/`groupby`, numeric for mean = proportion). Worth pointing out.
- Only 54 rows — `head()` shows a meaningful chunk of the data immediately.
- Dividing by a frequency column may produce `NaN` or `inf` if the row has zero frequency for that food. Suggest `df.dropna(subset=["EGGSFREQ"])` before deriving the per-serving ratio, or a `.replace([float('inf')], pd.NA)` afterward.

---

### Dataset 13

**Source:** Kaggle 80 Cereals.

**Shape:** 77 rows × 16 columns.

**Suggested columns:**
- numeric: `calories`, `protein`, `fat`, `sodium`, `fiber`, `carbo`, `sugars`, `potass`, `vitamins`, `shelf`, `weight`, `cups`, `rating`.
- categorical: `mfr` (manufacturer code: A, G, K, N, P, Q, R), `type` (C/H for cold/hot).
- plot: `mfr` (X) vs average `rating` (Y), via `groupby`.
- derived column: `df["calories_per_cup"] = df["calories"] / df["cups"]` or `df["sugar_per_serving"] = df["sugars"] * df["weight"]`.

**Quirks:**
- `mfr` codes are single letters; the team will need to look up which letter is which manufacturer (or just treat them as labels). Don't help unless asked.
- `name` is unique per row — fine for sort/filter but not for `value_counts` or `groupby`.
- A few cereals have negative values in `potass` and `sugars` (they represent missing data in the original). Watch for student confusion.

---

### Dataset 14

**Source:** Kaggle Starbucks drink menu (expanded).

**Shape:** 242 rows × 18 columns.

**Suggested columns:**
- numeric: `Calories`, `Total Fat (g)`, `Sodium (mg)`, `Total Carbohydrates (g)`, `Sugars (g)`, `Protein (g)`, `Vitamin A (% DV)` — most of the nutrient columns.
- categorical: `Beverage_category`, `Beverage_prep` (size/preparation type — small cardinality).
- plot: `Beverage_category` (X) vs average `Calories` (Y), via `groupby`.
- derived column: `df["sugar_per_calorie"] = df["Sugars (g)"] / df["Calories"]` — proportion of calories from sugar.

**Quirks:**
- **Several column names contain leading or trailing whitespace** (e.g. `" Total Fat (g)"`, `"Trans Fat (g) "` with a trailing space). Students will get `KeyError` unless they copy-paste exact strings from `df.columns`. Strongly suggest they print `df.columns.tolist()` first and copy values verbatim.
- Some "numeric" columns are sneaky strings — e.g. `Caffeine (mg)` contains `"Varies"` for some rows. Suggest `pd.to_numeric(df["Caffeine (mg)"], errors="coerce")` if they want to use that column.

---

### Dataset 15

**Source:** Kaggle Groundhog Day predictions vs actual February temperatures.

**Shape:** 132 rows × 10 columns.

**Suggested columns:**
- numeric: `February Average Temperature`, `February Average Temperature (Northeast)`, `February Average Temperature (Midwest)`, `February Average Temperature (Pennsylvania)`, and the corresponding `March Average Temperature` columns. Verify exact column names from `df.columns` since some may have spaces.
- categorical: `Punxsutawney Phil` (the prediction column — small set of values like "Full Shadow", "No Shadow", "Partial Shadow").
- plot: `Punxsutawney Phil` (X) vs average `February Average Temperature` (Y), via `groupby`.
- derived column: `df["feb_to_mar_change"] = df["March Average Temperature"] - df["February Average Temperature"]`.

**Quirks:**
- ~5% missing values across the temperature columns — meaningful work for Task 3. Encourage `dropna()` for the specific columns being used in a step rather than over the whole frame.
- The whole dataset is a setup for "does Phil's prediction correlate with actual February temps?" Strong narrative for the Detective Report.
- `Year` is a numeric column but using it for `describe`/`sort` mostly gives boring "1887 to 2018" output. Push them toward the temperature columns.
TA_DATASETS_EOF
```

- [ ] **Step 2: Verify per-dataset structure**

```bash
grep -c '^### Dataset ' content/ta_cheatsheet.md
grep -E '^### Dataset ' content/ta_cheatsheet.md
grep -c '^\*\*Source:\*\*' content/ta_cheatsheet.md
grep -c '^\*\*Shape:\*\*' content/ta_cheatsheet.md
grep -c '^\*\*Suggested columns:\*\*' content/ta_cheatsheet.md
grep -c '^\*\*Quirks:\*\*' content/ta_cheatsheet.md
```
Expected:
- First count: `15`.
- Second command lists `### Dataset 1` through `### Dataset 15` in numeric order.
- Each of the four `**Field:**` counts: `15`.

- [ ] **Step 3: Commit**

```bash
git add content/ta_cheatsheet.md
git commit -m "add per-dataset half to TA cheatsheet (15 entries with source, shape, columns, quirks)"
```

---

### Task 9: Final cross-check (verification only — no commit)

**Files:** none modified.

- [ ] **Step 1: All deliverables in place**

```bash
ls -la content/secret_dataset_2.csv content/secret_dataset_11.csv content/secret_dataset_12.csv content/notebook.ipynb content/student_cheatsheet.md content/ta_cheatsheet.md
```
Expected: all six files listed, all non-empty.

- [ ] **Step 2: Notebook is 10 cells in correct order**

```bash
python3 -c "
import json
nb = json.load(open('content/notebook.ipynb'))
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
assert len(nb['cells']) == 10
for i,(g,w) in enumerate(zip(heads, expected)):
    assert g == w, f'cell {i}: got {g!r}, expected {w!r}'
print('notebook ok')
"
```
Expected output: `notebook ok`.

- [ ] **Step 3: Preprocessed datasets all load without encoding errors**

```bash
python3 -c "
import csv
for n in (2, 11, 12):
    fn = f'content/secret_dataset_{n}.csv'
    with open(fn, encoding='utf-8') as f:
        rows = list(csv.reader(f))
    print(f'{fn}: {len(rows)-1} data rows, {len(rows[0])} cols')
print('preprocessed datasets ok')
"
```
Expected: 3 lines like `content/secret_dataset_<n>.csv: <N> data rows, <C> cols`, then `preprocessed datasets ok`. Specifically: dataset 2 has 16 cols, dataset 11 has 6 cols, dataset 12 has 15 cols.

- [ ] **Step 4: Cheatsheet structures match spec**

```bash
echo '=== student cheatsheet sections ==='
grep -E '^## ' content/student_cheatsheet.md
echo '=== TA cheatsheet per-task tasks ==='
grep -E '^## Task ' content/ta_cheatsheet.md
echo '=== TA cheatsheet datasets ==='
grep -c '^### Dataset ' content/ta_cheatsheet.md
```
Expected:
- Student cheatsheet: 6 `## ` sections in order (Loading data, Inspecting, Cleaning, Building, Analyzing, Plotting).
- TA per-task: 9 task headings in order (Task 1 through Task 9).
- TA datasets count: `15`.

- [ ] **Step 5: No commit needed for this verification-only task.**
