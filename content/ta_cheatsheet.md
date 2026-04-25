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
