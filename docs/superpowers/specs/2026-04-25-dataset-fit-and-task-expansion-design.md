# Dataset Fit and Task Expansion — Design

## Context

The workshop ships fifteen `secret_dataset_<n>.csv` files (one per team). The student notebook (`content/notebook.ipynb`) currently has six task cells designed around datasets with multiple numeric columns. Audit of the fifteen datasets revealed:

- **9 are clean numeric-heavy fits**: 4, 6, 7, 8, 9, 10, 13, 14, 15.
- **1 has minor format issues** (commas/percent signs in numeric columns) but is workable as-is: 1.
- **3 are survey-style with mostly categorical columns and very few numerics**: 2, 3, 5. They are not broken, but they don't fit the current task set — `describe`, `sort_values`, and the seaborn plot all assume numeric columns.
- **2 are unusable as-is**: 11 (every numeric cell is the BLS suppression code `**`, so pandas reads zero numerics) and 12 (1093 columns — `head()` is unreadable, students don't know where to start).

Separately, the workshop runs **2 hours** with **paired entry-level students**. The current six tasks are too thin to occupy that window for a strong pair, and the survey-shaped datasets above need additional task types regardless.

## Goals

1. Make every one of the 15 datasets usable for every task in the notebook without TA-side intervention beyond hints.
2. Expand the notebook from 6 tasks to 9 tasks, all entry-level, that work across both numeric-heavy and survey-style data shapes — sized to fill the 2-hour window for a pair.
3. Update the student cheatsheet to cover the new functions.
4. Update the TA cheatsheet with both per-task guidance and per-dataset guidance so a walking TA can anticipate team-specific stumbles.

## Non-goals

- Not converting cell 10 (Detective Report) from code to markdown cell type. Previously flagged as a separate fix.
- Not modifying README.md, requirements.txt, or the design/plan files.
- Not preserving copies of the original CSVs we rewrite — git history holds them.
- Not introducing pandas operations beyond entry-level: no merging, no reshaping, no advanced statistics, no time series, no `apply` with custom functions.

## Approach summary

Three coordinated changes:

1. **Preprocess three problem datasets** so they load cleanly with default `pd.read_csv()`.
2. **Add three new task cells** to the notebook (`value_counts`, derive a new column, `groupby` mean) that work on any dataset shape.
3. **Update both cheatsheets** to cover the new tasks, and split the TA cheatsheet into a per-task half and a per-dataset half.

---

## 1. Dataset preprocessing

### `secret_dataset_2.csv` — Star Wars survey

**Problems:** latin-1 encoding (default `pd.read_csv` raises `UnicodeDecodeError`); two-row header (sub-header sits in row 1 of data, garbling everything); 38 columns, most of which are character-favorability matrices irrelevant to the workshop tasks.

**Fix:** rewrite the file as utf-8 with a single flat header. Subset to 16 columns:

| New column name | Source |
| --- | --- |
| `RespondentID` | as-is |
| `has_seen_franchise` | "Have you seen any of the 6 films in the Star Wars franchise?" |
| `is_fan_starwars` | "Do you consider yourself to be a fan of the Star Wars film franchise?" |
| `rank_ep1_phantom_menace` | from "Please rank the Star Wars films…" block |
| `rank_ep2_clones` | same |
| `rank_ep3_sith` | same |
| `rank_ep4_new_hope` | same |
| `rank_ep5_empire` | same |
| `rank_ep6_jedi` | same |
| `shot_first` | "Which character shot first?" |
| `is_fan_startrek` | "Do you consider yourself to be a fan of the Star Trek franchise?" |
| `gender` | as-is |
| `age` | as-is |
| `household_income` | as-is |
| `education` | as-is |
| `region` | "Location (Census Region)" |

The 6 rank columns are coerced to numeric (`1`–`6` integers; missing where the respondent didn't rank). Everything else is text. Rows where the respondent answered "No" to `has_seen_franchise` are kept (their rank columns will be NaN — useful for Task 3).

### `secret_dataset_11.csv` — Librarians by MSA

**Problem:** every numeric cell holds `**` (BLS suppression code for confidentiality), so pandas reads all four "numeric" columns as text.

**Fix:** rewrite the file replacing every `**` with an empty cell. After the rewrite, `pd.read_csv()` parses `tot_emp`, `emp_prse`, `jobs_1000`, `loc_quotient` as floats with NaN where suppressed. **Risk:** if the rewrite leaves too many fully-NaN rows, the team has too little to analyze. Mitigation: after preprocessing, count rows that have at least one non-NaN numeric — if fewer than ~80 rows, drop fully-suppressed rows from the rewrite and document the remaining count in the TA per-dataset section.

### `secret_dataset_12.csv` — Nutrition study

**Problem:** 1093 columns. `df.head()` is unreadable; students cannot pick a column meaningfully.

**Fix:** rewrite the file with a 15-column subset:

```
ID, cancer, diabetes, heart_disease, belly,
ever_smoked, currently_smoke,
cat, dog,
EGGSFREQ, EGGSQUAN,
COLDCEREALFREQ, COLDCEREALQUAN,
WHITERICEFREQ, WHITERICEQUAN
```

Eight binary categorical columns (good for `value_counts`), six numeric food columns (good for `describe` / `groupby`), and `ID`.

### Datasets we are NOT preprocessing

- **#1 (Pulitzer):** the "Daily Circulation" columns have commas; "Change in Daily Circulation" has `+`/`-`/`%`. Pandas reads them as text, but the team still has 3 real numeric columns (the Pulitzer prize counts) — enough for every task. TA cheatsheet flags this so curious teams can clean the columns themselves as a stretch.
- **#3 (Thanksgiving) and #5 (Weather):** survey data, mostly categorical, very few numerics. Once the new `value_counts` and `groupby` tasks land, these datasets become first-class — no preprocessing needed. TA cheatsheet flags that `describe` / `sort` / numeric-only-plot will be thin for these teams; they should lean into the new tasks.

---

## 2. Notebook task expansion

### Final task sequence

Cell ids in parentheses where the cell already exists.

| Cell | Task | Status |
| --- | --- | --- |
| 1 (`d66ddc83`) | Load | unchanged |
| 2 (`af606d09`) | Head | unchanged |
| 3 (`3871cee3`) | Missing values | unchanged |
| 4 (`564f921a`) | `describe()` — numeric summary | unchanged |
| 5 (new) | **`value_counts()` — categorical summary** | new |
| 6 (`2bc1092b`) | Sort & filter | renumber TASK 5 → TASK 6 in cell text |
| 7 (new) | **Derive a new column** | new |
| 8 (new) | **`groupby` mean** | new |
| 9 (`a1e4b985`) | Plot | renumber TASK 6 → TASK 9 in cell text |
| 10 (`6d3ccf1a`) | Detective report (markdown content in code cell) | unchanged |

Three new cells are inserted between existing cells. Existing cell IDs are preserved (only their internal "TASK N" numbering text changes for cells 6 and 9). New cells get fresh ids assigned by `NotebookEdit`.

### New task cell content

Each new cell is comments-only (matches the existing strip pattern). Each comment block names neither the function nor a hint that gives away the answer; instead it points to the cheatsheet section where the function lives.

**Task 5 — `value_counts`**

> Pick a categorical column from your dataset (look at the columns from Task 2 — categorical means columns whose values are categories, words, or labels rather than numbers).
>
> Your job:
> - Count how many times each value appears in that column.
> - Look at the result and discuss with your team: which values are most common? Which are rare? Are there any surprises?
>
> Check the cheatsheet under "Inspecting" for the function you need.

**Task 7 — Derive a new column**

> Sometimes the column you want to analyze isn't already in the dataset — you have to build it. For example, if you have separate `beer_servings` and `wine_servings` columns, you might want a `total_servings` column.
>
> Your job:
> - Pick one or two existing columns from `df`.
> - Create a NEW column whose values are computed from those existing column(s). Examples:
>   - a sum: `df["total"] = df["a"] + df["b"]`
>   - a difference: `df["diff"] = df["a"] - df["b"]`
>   - a ratio: `df["ratio"] = df["a"] / df["b"]`
> - Check it worked by running `df.head()` again — your new column should appear at the right.
>
> Check the cheatsheet under "Building" for the syntax pattern.

**Task 8 — `groupby` mean**

> Now you'll combine a category column with a numeric column to find which categories have the highest/lowest averages.
>
> Your job:
> - Pick a categorical column and a numeric column from `df`.
> - Compute the average of the numeric column for each category in the categorical column.
> - Discuss with your team: which category has the highest average? Which is lowest? Anything unexpected?
>
> Check the cheatsheet under "Analyzing" for the function you need.

### Existing cells affected

- Cell 6 (`2bc1092b`, sort/filter): change `TASK 5` to `TASK 6` in the comment heading.
- Cell 9 (`a1e4b985`, plot): change `TASK 6` to `TASK 9` in the comment heading.

No other content in those cells changes.

---

## 3. Cheatsheet updates

### Student cheatsheet (`content/student_cheatsheet.md`)

Three additions, one in an existing section, one new section, one in an existing section:

- **Inspecting** — append: `df["col_name"].value_counts() — count how many times each unique value appears in a column.`
- **Building** — *new section* between "Cleaning" and "Analyzing":
  - `df["new_col"] = df["a"] + df["b"]` — create a new column that's the sum of two existing columns.
  - `df["new_col"] = df["a"] / df["b"]` — create a new column that's a ratio.
  - `df["new_col"] = df["a"] - df["b"]` — create a new column that's a difference.
- **Analyzing** — append: `df.groupby("cat_col")["num_col"].mean() — average of "num_col" for each category in "cat_col".`

Section order after change: Loading data → Inspecting → Cleaning → Building → Analyzing → Plotting.

### TA cheatsheet (`content/ta_cheatsheet.md`)

The file is restructured into two top-level halves.

**Top of file:** preamble unchanged.

**Half 1 — Per-task guidance.** Existing 6 sections renumbered + 3 new sections inserted. Final section list:

1. Task 1 — Load the dataset *(unchanged)*
2. Task 2 — First few rows *(unchanged)*
3. Task 3 — Missing values *(unchanged)*
4. Task 4 — Descriptive statistics *(unchanged)*
5. Task 5 — Count categories *(new)*
6. Task 6 — Sort and filter *(was Task 5; only the heading number changes — content unchanged)*
7. Task 7 — Derive a new column *(new)*
8. Task 8 — Group and average *(new)*
9. Task 9 — Plot *(was Task 6; only the heading number changes — content unchanged)*

Each new section follows the existing template: **Answer** (Python), **Common pitfalls** (3–4 bullets), **Hints to give** (3 Socratic prompts that don't name the function).

**Half 2 — Per-dataset guidance.** New top-level heading `## Datasets`, followed by fifteen subsections (`### Dataset 1` through `### Dataset 15`). Each subsection contains:

- **Source** — one line: name and origin (e.g. "FiveThirtyEight Star Wars survey").
- **Shape** — rows × cols after any preprocessing.
- **Suggested columns** — explicit picks for the tasks where students choose:
  - a numeric column for sort/filter and for the numeric side of `groupby`
  - a categorical column for `value_counts` and the grouping side of `groupby`
  - two columns for plot
  - two columns for the derived column
- **Quirks** — bullet list of dataset-specific gotchas: missing-value patterns, surprising column dtypes, intuition pitfalls (e.g. "for Star Wars the rank columns invert intuition — `1` is *favorite*, not *least liked*").

The fifteen entries are written individually — no template fill-in. Concrete examples of quirks per dataset:

- **Dataset 1 (Pulitzer):** circulation columns look numeric but are text (commas + `%`). Curious teams can investigate with `str.replace`; default-flow teams should pick from the 3 prize-count columns.
- **Dataset 2 (Star Wars):** rank columns are numeric (1–6) but `1` is favorite — sort `ascending=True` to get "best ranked." Many NaN in rank columns from non-watchers — worth dropping or filtering.
- **Dataset 3 (Thanksgiving):** only one numeric column; lean heavily on `value_counts` and `groupby`. The derive task is awkward — accept simple recodes or string concatenations.
- **Dataset 5 (Weather):** same shape as Thanksgiving — only `RespondentID` is numeric. Same TA approach.
- **Dataset 11 (Librarians):** lots of NaN after preprocessing. `dropna()` early. Numeric columns are MSA-level metrics — `loc_quotient` is the most interesting to sort/group on.
- **Dataset 12 (Nutrition):** binary 0/1 columns can be used both as categoricals (for `value_counts`/`groupby`) and as numerics (mean of a 0/1 column = proportion).
- ... and analogous entries for the other nine datasets, each grounded in the columns we observed during the audit.

The TA cheatsheet preamble is updated to mention both halves and to direct TAs: "When a team is stuck on *what to do*, look in Per-task guidance. When a team is stuck on *what their data is*, look in Datasets."

---

## File-by-file change summary

| File | Kind | Net effect |
| --- | --- | --- |
| `content/secret_dataset_2.csv` | rewrite | utf-8, 16-column subset, single flat header |
| `content/secret_dataset_11.csv` | rewrite | `**` cells replaced with empty cells |
| `content/secret_dataset_12.csv` | rewrite | 15-column subset |
| `content/notebook.ipynb` | modify | three new cells inserted (positions 5, 7, 8 in the new ordering); two existing cells get heading number updated |
| `content/student_cheatsheet.md` | modify | one new section ("Building"), three new bullet entries |
| `content/ta_cheatsheet.md` | rewrite | two-part structure: nine per-task sections (renumbered + 3 new), fifteen per-dataset sections |

## Risks summary

- **#11 over-suppression**: if the `**` → NaN rewrite leaves the dataset too thin to analyze, fall back to dropping fully-suppressed rows during the rewrite and note remaining row count in the TA per-dataset entry. Verified during implementation, not deferred.
- **Survey datasets and the derive task**: #3 and #5 have only one numeric column, so the canonical `a + b` derivation pattern doesn't fit. TA per-dataset guidance for those teams documents acceptable substitutes (e.g. `df["age_group"] = df["A. Age"]` for a recode, or string concat).
- **2-hour fit is an estimate**: 9 tasks × ~10–13 min/task per pair ≈ 90–120 min. Strong pairs may finish early; the Detective Report cell absorbs slack. Slow pairs may not finish — TA cheatsheet should note that Tasks 5 and 8 are the highest-leverage stops and Task 7 can be skipped under time pressure.

## Out of scope (explicit)

- Cell 10 type change (code → markdown). Tracked separately.
- Adding new datasets, removing existing datasets, renaming dataset files.
- Statistical analysis beyond mean/sort/filter/group: no correlation, no regression, no hypothesis testing.
- Multi-step pipelines, function definitions, or imports beyond what's already in cell 1.
