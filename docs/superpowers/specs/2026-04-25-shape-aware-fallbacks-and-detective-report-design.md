# Shape-Aware Fallbacks and Detective Report — Design

## Context

We walked all 15 secret datasets through all 9 notebook tasks (analytic dry-run — see chat history). Result: 8 datasets work cleanly across every task; 4 datasets have shape-specific gaps that the current notebook doesn't address.

| Dataset | Tasks affected | Gap |
| --- | --- | --- |
| #3 Thanksgiving | T4, T6, T7 | Only `RespondentID` is numeric — `describe`/`sort_values` are thin, derive needs a non-numeric recipe |
| #5 Weather | T4, T6, T7 | Same shape as #3 |
| #4 Unisex names | T5, T8 | No usable categorical column (`name` is unique-per-row) |
| #7 Alcohol | T5, T8 | Same shape as #4 (`country` is unique-per-row) |

The TA cheatsheet currently covers some of this (e.g. `.size()` for #3 and #5 in Task 8) but the notebook itself gives no guidance to teams whose data doesn't fit the canonical task shape. Workshop runs ~7 teams concurrently with limited TAs, so notebook-level fallbacks are higher leverage than TA-side instructions.

Separately, the existing **Detective Report** cell has 4 fill-in prompts that assume an "average value" exists and uses an "is this El Paso weather?" framing that no longer fits — datasets are recognizable from column names within seconds. The cell's prompts also don't reflect the three new tasks (`value_counts`, derived column, `groupby`).

The user has manually converted cell 9 (Detective Report) from code-cell to markdown-cell since the previous design cycle, so the type-bug is no longer blocking; new content can use proper markdown formatting.

## Goals

1. Make every notebook task gracefully handle the four problem dataset shapes via inline fallback hints — students get unstuck without needing TA intervention.
2. Replace the Detective Report cell with a 6-prompt synthesis exercise that works for any dataset shape and rewards work done in the new tasks.
3. Add the small set of cheatsheet entries the new fallbacks rely on (`.size()` and the boolean-derive pattern).
4. Touch up the TA cheatsheet entries for datasets #4 and #7 to cite the new boolean-derive recipe.

## Non-goals

- No new datasets, no dataset removals, no further preprocessing.
- No new notebook tasks beyond the 9 we have.
- No restructuring of existing notebook cell content beyond the additions described below.
- No changes to README.md or requirements.txt.
- No statistical operations beyond `.size()` and a boolean comparison.

## Approach

Three coordinated edits:

1. **Append fallback hints to four existing notebook task cells** (Tasks 4, 6, 7, 8). The hints are short (1–4 lines each), placed at the end of each cell so the canonical task remains the lead.
2. **Replace the Detective Report cell content** with the 6-prompt markdown version. Cell type stays markdown.
3. **Add two student cheatsheet entries** (`.size()` and boolean-derive) and **two TA cheatsheet column-suggestion lines** (datasets 4 and 7).

---

## 1. Notebook fallback hints

Each addition is appended to the existing cell — none of the existing comments are removed or rewritten.

### Task 4 — `describe()` cell (id `564f921a`)

After the existing discussion bullets, append a separator line and:

```
# If your dataset has only one numeric column (or none), you'll get a thin
# describe() output — that's OK. Your insights will come from Tasks 5 and 8
# instead.
```

### Task 6 — Sort & filter cell (id `2bc1092b`)

After the existing Sub-task B and the cheatsheet pointer, append:

```
# If your dataset has no useful numeric column to sort or filter on, skip
# this task and move on — Tasks 7, 8, and 9 will still give your team
# plenty to do.
```

### Task 7 — Derive a new column cell (id `1b0312dc`)

After the existing example shapes block (the sum/diff/ratio examples), insert a new "or" branch BEFORE the line that says "Confirm it worked by running df.head() again":

```
#   - Or, if your dataset doesn't have two numeric columns to combine,
#     derive a True/False category from a single column instead:
#         df["new_cat"] = df["a"] > value
#     This gives you a fresh category column you can use in Tasks 8 and 9.
```

The "Confirm it worked..." line and the cheatsheet pointer remain in place after this insertion.

### Task 8 — `groupby` mean cell (id `c11cb1b4`)

After the existing discussion bullets, append:

```
# If .mean() doesn't make sense for your data (for example, your only
# numeric column is an ID), use .size() instead — it counts how many rows
# are in each group, which is just as interesting:
#     df.groupby("cat_col").size()
```

---

## 2. Detective Report cell replacement

Cell `88fd3022` (markdown). Full replacement source:

```markdown
### Team Detective Report

Now that you've explored your dataset, fill out this report with your team. Each prompt asks about something you produced in the tasks above — go back to the relevant cell to grab numbers or column names if you need to.

1. **The dataset.** What is your data about? Which of your columns made it obvious?

2. **The most surprising number** (from Task 4 or Task 8). Pick one number from `describe()` or `groupby` that surprised you. What was it, and why?

3. **The biggest split** (from Task 5 or Task 8). Which value or group came out on top — the most common category, or the highest/lowest group average?

4. **The strongest pattern in your plot** (from Task 9). What does your chart show — a trend, a difference between groups, an outlier? One sentence.

5. **One question your dataset CAN'T answer.** What did you wish you could analyze, but the columns you have don't support?

6. **Your team's headline.** Write a single sentence that captures the most interesting thing you found, as if it were a news article headline.
```

Each numbered prompt uses bold for the prompt label and backticks for the function names so it renders cleanly.

---

## 3. Cheatsheet additions

### Student cheatsheet (`content/student_cheatsheet.md`)

**Building** — append one bullet at the bottom of the section:

```
- `df["new_col"] = df["a"] > value` — create a True/False column you can use as a category for value_counts or groupby.
```

**Analyzing** — append one bullet at the bottom of the section:

```
- `df.groupby("cat_col").size()` — count how many rows are in each category (alternative to `.mean()` when there's no useful numeric column).
```

No section reordering, no other changes.

### TA cheatsheet (`content/ta_cheatsheet.md`)

**Dataset 4 (unisex names)** — within the existing `**Suggested columns:**` block, replace the categorical line with:

```
- categorical: `name` is unique-per-row, so this team needs to derive a category. Show them the boolean-derive recipe: `df["male_dominant"] = df["male_share"] > df["female_share"]`. Use the new column for value_counts and groupby.
```

**Dataset 7 (alcohol)** — within the existing `**Suggested columns:**` block, replace the categorical line with:

```
- categorical: `country` is unique-per-row, so this team needs to derive a category. Show them the boolean-derive recipe: `df["heavy_drinker"] = df["total_litres_of_pure_alcohol"] > df["total_litres_of_pure_alcohol"].median()`. Use the new column for value_counts and groupby.
```

Existing **Quirks** bullets in those entries are unchanged. No other TA cheatsheet edits — entries for #3 and #5 already mention `.size()`.

---

## File-by-file change summary

| File | Kind | Net effect |
| --- | --- | --- |
| `content/notebook.ipynb` | modify | append fallback comments to 4 cells, replace Detective Report markdown source |
| `content/student_cheatsheet.md` | modify | one new bullet in Building, one new bullet in Analyzing |
| `content/ta_cheatsheet.md` | modify | one categorical-line replacement in each of Dataset 4 and Dataset 7 entries |

## Risks

- **Notebook fallback noise**: appending fallback hints lengthens cells slightly. Mitigation: place fallbacks at the end of each cell behind the canonical task, so a team whose dataset doesn't need the fallback can ignore it. Lines are commented and short.
- **Detective Report length**: 6 prompts may be too many for slow pairs. Mitigation: prompts 5 and 6 are explicitly synthesis prompts (no fresh analysis required) — slow pairs can write a one-liner each in the last 5 minutes.
- **Survey-team confidence**: even with fallbacks, a #3/#5 team that finishes Task 4 with "describe is thin" may feel they're behind. Mitigation: the fallback message frames the thin output as expected and points forward to Tasks 5 and 8 as the real source of insight.

## Out of scope (explicit)

- Cell-type fix for Detective Report — already done by user.
- Adding/removing datasets.
- Adding more notebook tasks.
- Changing the existing 9 task headings or their canonical content.
- Anything outside `content/`.
