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
