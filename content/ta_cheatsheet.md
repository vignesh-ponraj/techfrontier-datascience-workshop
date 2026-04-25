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
