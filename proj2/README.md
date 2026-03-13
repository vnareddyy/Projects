# PRT Feedback Analysis — Data Science Practicum

Analysis of resident survey data to support decisions on **adding**, **eliminating**, or **changing** PRT bus routes based on resident feedback.

---

## How to Run the Code

### Prerequisites

- **Python 3.8+** (3.9 or 3.10 recommended)
- **Jupyter** (e.g. JupyterLab, Jupyter Notebook, or VS Code with Jupyter extension)

### 1. Install Dependencies

From the project directory:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
```

Or with a requirements file:

```bash
pip install -r requirements.txt
```

*(See [Requirements](#requirements) below for a sample `requirements.txt`.)*

### 2. Set the Data Path

Open `prt_feedback_analysis.ipynb` and find the cell that sets:

```python
DATA_PATH = "/Users/vinaynareddy/Documents/prt/prt_survey_data.csv"
```

- If your CSV is elsewhere, change `DATA_PATH` to the full path to `prt_survey_data.csv`.
- To use a path relative to the notebook (e.g. `../prt/prt_survey_data.csv`), you can set:
  ```python
  import os
  DATA_PATH = os.path.join(os.path.dirname(os.path.abspath(".")), "prt_survey_data.csv")
  ```
  and place the CSV in the same folder as the notebook, or adjust the path accordingly.

### 3. Run the Notebook

**Option A — Jupyter in browser**

```bash
cd /Users/vinaynareddy/Documents/lastsem/proj2
jupyter notebook prt_feedback_analysis.ipynb
```

Then in the browser: **Cell → Run All**.

**Option B — JupyterLab**

```bash
cd /Users/vinaynareddy/Documents/lastsem/proj2
jupyter lab prt_feedback_analysis.ipynb
```

Then: **Run → Run All Cells**.

**Option C — VS Code / Cursor**

1. Open `prt_feedback_analysis.ipynb`.
2. Select a Python kernel (e.g. your conda/venv).
3. Use **Run All** in the notebook toolbar (or run cells one by one with Shift+Enter).

**Option D — Run from terminal (no browser)**

Execute the entire notebook from the command line. Outputs and plots are saved in the notebook file.

```bash
cd /Users/vinaynareddy/Documents/lastsem/proj2
jupyter nbconvert --to notebook --execute prt_feedback_analysis.ipynb
```

This runs all cells and overwrites the notebook with one that includes the results. To keep the original notebook unchanged and write results to a new file:

```bash
jupyter nbconvert --to notebook --execute prt_feedback_analysis.ipynb --output prt_feedback_analysis_executed.ipynb
```

To run without saving outputs back into the notebook (execute only):

```bash
jupyter execute prt_feedback_analysis.ipynb
```

*(If `jupyter execute` is not found, use the `nbconvert` command above.)*

### 4. Expected Runtime

- Full run (all cells): typically **under a minute** on a modern laptop for ~1,700 rows.
- If you see `ModuleNotFoundError`, install the missing package (e.g. `pip install seaborn`).

---

## Requirements

Create a `requirements.txt` in the same folder as the notebook with:

```
pandas>=1.3.0
numpy>=1.20.0
matplotlib>=3.4.0
seaborn>=0.11.0
scikit-learn>=1.0.0
jupyter>=1.0.0
```

Then run: `pip install -r requirements.txt`.

---

## Thought Process: Why Each Step Was Performed

Below is the reasoning behind each section of the analysis, so you can see *why* each step was done and how it supports route decisions.

---

### 1. Setup and Load Data

**Why:** Establish a single place for imports and configuration so the rest of the notebook is readable and maintainable. Loading the raw CSV and inspecting shape/columns confirms we have the right file and variable names before any analysis.

**Why these libraries:**  
- **pandas/numpy** for tables and numeric operations.  
- **matplotlib/seaborn** for clear, reproducible visualizations.  
- **sklearn** for encoding, train/test splits, Random Forest, and TF-IDF.  
- **re** for cleaning and extracting route numbers from free text.

---

### 2. Exploratory Data Analysis (EDA)

**Why:** Survey columns have long, unwieldy names and mixed types. We need to:

- **Rename columns** so code stays readable and we can reference fields like `route_1` and `impact_peak_reduction` without errors.
- **Coerce impact columns to numeric** because they are 1–5 scales; treating them as numbers is required for averages and modeling.
- **Inspect missing values** to know where we can safely aggregate (e.g. impact by route) and where we might bias results by dropping rows.

This step ensures the dataset is in a consistent, analysis-ready form before any business logic.

---

### 3. Route Usage Analysis

**Why:** To decide which routes to add, keep, or cut, we first need to know **which routes residents actually use** and how strongly they rely on them.

- **Clean route names** (strip whitespace, treat blanks as missing) so “54” and “54 ” are not counted separately.
- **Weighted usage** (1st = 3, 2nd = 2, 3rd = 1) reflects that the “most often” route matters more than the third; simple counts would over-weight occasional use.
- **Ranking and bar chart** give a quick, defensible view of high-use routes that should be treated carefully when considering elimination or major changes.

---

### 4. Impact Scores — Who Is Hurt vs. Helped?

**Why:** The survey asks three impact questions (peak reduction, direct services, off-peak expansion) on a 1–5 scale. We need a single, interpretable measure of “overall impact” and a way to label who is positive vs negative.

- **Composite impact** = average of the three 1–5 scores. One number per respondent makes it easy to compare groups (e.g. by route or age) and to build models later.
- **Sentiment labels** (positive / neutral / negative) turn the scale into categories that match how we talk about “supporters” vs “concerned” residents and allow classification models.
- **Impact by primary route** (mean impact and % negative for each route_1) answers: “If we change or cut route X, how negatively affected are the people who rely on it most?” Routes with high usage and high negative impact are priority candidates for “reconsider eliminating” or “modify instead of cut.”
- **Plots** (sentiment distribution, impact distribution) communicate the overall mix of opinions to stakeholders who may not look at tables.

---

### 5. Demographics and Impact

**Why:** Equity and targeting: different age groups and trip patterns may be affected differently. If we only look at averages, we can miss groups that are disproportionately hurt.

- **Impact by age** shows whether older or younger riders are more concerned; this can inform communication and targeted mitigations.
- **Impact by last trip** (Today / past week / past month / etc.) tests whether **recent riders** (who have current experience) feel differently from those who rarely ride; their opinions are often given more weight in service design.

---

### 6. Text Feedback — Reasons for Negative vs Positive Impact

**Why:** The 1–5 scales tell us *that* people are negative or positive, but not *why*. The open-ended fields (“reason for negative impact,” “excited about,” “additional comments”) contain the actual arguments and route names.

- **Count non-empty text** so we know how much qualitative data we have and whether conclusions from text are representative.
- **Word frequency in negative reasons** surfaces recurring themes (e.g. “slower,” “transfers,” “walking”) that summarize resident concerns without reading every response.
- **Route mentions in negative text** (via a simple pattern for route IDs like 54, 28X, P71) identify which routes are **explicitly complained about**—complementing the numeric “impact by route” from Section 4.
- **Route mentions in positive/excited text** identify which **new or changed routes** residents want (e.g. South Hills–Oakland direct, O47); this directly supports “add or expand” recommendations.

---

### 7. Route-Level Recommendations (Add / Eliminate / Change)

**Why:** We need one place that combines **usage**, **measured impact**, and **qualitative mentions** so we can rank routes for action.

- **Merge** usage (weighted count), impact (mean, % negative), and mention counts (negative and positive) into a single route-level table.
- **Risk score** = weighted_usage × mean_impact + 2 × mentions_in_negative. This prioritizes routes that are both heavily used and highly negative, and where many people mention the route in negative comments. High risk score → “reconsider eliminating” or “change, don’t cut.”
- **Sort by risk_score** yields an ordered list for “routes to reconsider eliminating.” The “excited” mentions list gives “routes to add or expand.”

---

### 8. Machine Learning — Predicting Impact Sentiment

**Why:** We want to know **which observable factors** (route, recency of use, age, gender, income) best predict whether someone will be positive, neutral, or negative. That tells us which levers matter for communication and targeting.

- **Features:** route_1, last_trip, age, gender, income (label-encoded). These are available for most respondents and are actionable (e.g. “users of route X,” “recent riders”).
- **Target:** impact_sentiment (positive / neutral / negative). We predict sentiment from demographics and usage, not from the text fields.
- **Random Forest** handles mixed categorical inputs and gives **feature importance**; we interpret which variable (e.g. route_1 or last_trip) most drives sentiment. That supports statements like “primary route and recency of use are the strongest predictors of concern.”

---

### 9. Text-Based Sentiment (TF-IDF + Classifier)

**Why:** Open-ended text may contain signals that demographics and route choice do not (e.g. “bus stop moved,” “no service to Oakland”). A text-based model checks whether **wording** alone can predict negative vs non-negative sentiment.

- **Combine** reason_negative and additional_comments into one text field to maximize signal.
- **Filter** to responses with enough text so the model has something to learn from.
- **Binary target:** negative vs non-negative (positive + neutral) to keep the task simple and actionable.
- **TF-IDF** (unigrams and bigrams) converts text to a numeric representation; **Random Forest** then yields **feature importance** over terms. The top-weighted terms (e.g. “slower,” “transfers,” “eliminating”) summarize what language is associated with negative feedback and can be used to tag or prioritize comments in future surveys.

---

### 10. Summary and Recommendations

**Why:** Stakeholders need a short, narrative summary and clear next steps, not only plots and tables.

- **Summary stats** (total responses, % negative/positive, mean impact) give the headline numbers.
- **Tables** of “routes to reconsider eliminating” and “routes residents are excited about” are the direct outputs for planning (add / eliminate / change).
- **Written recommendations** tie the analysis back to decisions: e.g. use risk score to flag cuts to reconsider, use “excited” mentions to prioritize new service, use negative themes to prefer “change, don’t cut,” and use feature importance to focus on recent riders and primary-route users.

---

## Outputs You Can Use

- **Route usage ranking** — which routes are used most (weighted).
- **Impact by route** — mean impact and % negative per primary route.
- **Risk score table** — routes to **reconsider eliminating** (high use + high negative impact + many negative mentions).
- **Excited-route list** — routes/corridors to **add or expand**.
- **Feature importance** (demographics + route) — which factors best predict sentiment.
- **Top TF-IDF terms** — which phrases are most associated with negative feedback.

---

## File Structure

```
proj2/
├── README.md                    # This file (thought process + run instructions)
├── prt_feedback_analysis.ipynb  # Main analysis notebook
└── requirements.txt            # Optional: pip install -r requirements.txt
```

Data file (outside this folder in the example):

- `prt_survey_data.csv` — path set via `DATA_PATH` in the notebook.

---

## Troubleshooting

| Issue | What to do |
|-------|------------|
| `FileNotFoundError` for CSV | Set `DATA_PATH` in the notebook to the correct path to `prt_survey_data.csv`. |
| `ModuleNotFoundError` | Install the missing package (e.g. `pip install seaborn scikit-learn`). |
| Kernel not found in VS Code | Select a Python interpreter that has the required packages (e.g. conda/venv). |
| Plots not showing | In a cell, run `%matplotlib inline` (Jupyter) or ensure the notebook is in interactive mode. |
