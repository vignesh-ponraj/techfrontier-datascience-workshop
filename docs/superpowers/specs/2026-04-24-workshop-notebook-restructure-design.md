# Workshop Notebook Restructure — Design

## Context

`content/notebook.ipynb` is the introductory exercise for a Data Science workshop. The TA (the user) plans to assign each team a `secret_dataset_<n>.csv` and have them work through six tasks: load data, inspect, clean, describe, sort/filter, plot. Currently the notebook ships with both task descriptions and the worked Python code in each cell, which removes the discovery element.

## Goals

1. Convert `notebook.ipynb` into a task-only worksheet — students see what to do, not how.
2. Give students a standalone reference (`student_cheatsheet.md`) listing the pandas/seaborn functions they will need.
3. Give TAs a separate guide (`ta_cheatsheet.md`) with answer code, common pitfalls, and Socratic hints for walking the room.

## Non-goals

- Creating the `secret_dataset_*.csv` files (the user will source those separately).
- Editing `README.md` or `requirements.txt`.
- Changing the pedagogical sequence of the six tasks.

## Files affected

| Path | Status | Purpose |
| --- | --- | --- |
| `content/notebook.ipynb` | modified | Task-only worksheet |
| `content/student_cheatsheet.md` | new | Function reference card |
| `content/ta_cheatsheet.md` | new | TA guidance per task |

## Notebook strip plan

Cell 1 (imports) keeps its code as infrastructure. Cells 2–6 lose all Python and keep only descriptive comments. Cell 7 is markdown and is unchanged.

| Cell | Current code | After |
| --- | --- | --- |
| 1 imports | `import pandas as pd` … `import seaborn as sns` plus filename comment and `df = pd.read_csv("FILENAME.csv")` | Imports kept. The `df = pd.read_csv(...)` line is **stripped** — loading the dataset is the first student task. Comments expanded so students know they need to (a) find their team's file in the file browser, (b) call `read_csv` with that filename, (c) assign the result to `df`. |
| 2 head | `df.head()` | Comments only. Description tells the student to display the first 5 rows so they can see column names and data types. |
| 3 missing | `print(...)` lines + `df.isnull().sum()` | Comments only. Description: count missing values per column, then decide whether to fill or drop, and write the cleaning code. |
| 4 describe | `stats = df.describe()` + prints | Comments only. Description: generate descriptive statistics, then read the mean/max/std and reflect on what the numbers say about the data. |
| 5 sort/filter | `sorted_data = df.sort_values(...)` + filter example | Comments only. Two sub-tasks: (a) sort by a column they pick and show the top 5, (b) filter rows where a column exceeds its mean. |
| 6 plot | `plt.figure`, `sns.barplot`, `plt.title`, `plt.xticks`, `plt.show` | Comments only. Description: pick two columns, choose `barplot`/`lineplot`/`scatterplot` based on what makes sense, add a title, rotate x-ticks if needed, show the plot. |
| 7 report | markdown fill-in | Unchanged. |

Comments in stripped cells must be self-contained — a student reading only that cell should know what to produce, what column choices they need to make, and what the success criterion is.

## `student_cheatsheet.md` — shape

Terse one-line-per-function reference, no examples. Sections:

- **Loading data** — `pd.read_csv`
- **Inspecting** — `df.head`, `df.columns`, `df.shape`, `df.dtypes`
- **Cleaning** — `df.isnull().sum`, `df.fillna`, `df.dropna`
- **Analyzing** — `df.describe`, `df.sort_values`, boolean filtering pattern, `df["col"].mean`
- **Plotting** — `plt.figure(figsize=...)`, `sns.barplot`, `sns.lineplot`, `sns.scatterplot`, `plt.title`, `plt.xticks(rotation=...)`, `plt.show`

Each line is `function-signature — what it does`. No multi-line code blocks.

## `ta_cheatsheet.md` — shape

One section per task (Task 1 through Task 6, matching the notebook cells 1–6). Each section has three labeled subsections:

1. **Answer** — a fenced Python block showing one valid solution. For tasks where students must pick a column, use a generic placeholder like `"some_column"` and note the choice is dataset-dependent.
2. **Common pitfalls** — bulleted list of mistakes TAs are likely to see (e.g., wrong filename quoting, case-sensitive column names, forgetting `plt.show()`, treating `isnull().sum()` output as if it were the cleaned DataFrame).
3. **Hints to give** — bulleted list of Socratic prompts that guide without revealing the function name (e.g., *"What method gives you a quick statistical summary of every numeric column?"* rather than naming `.describe()`).

A short preamble at the top of the file reminds TAs that each team has a different `secret_dataset_<n>.csv`, so column names will vary.

## Risks and mitigations

- **Risk:** Stripped cells become too cryptic and students stall.
  **Mitigation:** Comments must be expanded relative to the current ones — descriptive enough that the cheatsheet is the only extra resource needed.
- **Risk:** TA cheatsheet's answer code uses a column name that doesn't exist in a team's secret dataset.
  **Mitigation:** Use placeholder column names in answer code and a one-line note at the top of `ta_cheatsheet.md`.

## Out of scope (explicit)

- Generating, downloading, or naming the secret datasets.
- Changes to the existing `content/data.csv` placeholder.
- Any change to deployment, CI, or `requirements.txt`.
