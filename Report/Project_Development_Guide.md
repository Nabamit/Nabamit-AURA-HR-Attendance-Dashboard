# Project Development Guide: Employee Attendance & Productivity Analytics

This guide provides a comprehensive, step-by-step tutorial on how to develop and execute the **Employee Attendance & Productivity Analytics** project from scratch. It explains how to structure the project folders, inspect and verify the raw data, execute data cleaning, run exploratory data analysis (EDA), perform rigorous statistical hypothesis testing, and transition findings into a dashboard report.

---

## 1. Project Folder Structure

To maintain a clean and reproducible data science workflow, the workspace is organized into five functional directories:

```text
Emp_Attendance/
├── Datasets/
│   ├── employee_attendance_bangalore_q1_2026.csv                          # Raw dataset
│   └── Employee-attendance-and-login-logout-data-bangalore_cleaned.csv    # Final cleaned dataset
├── Data Cleaning/
│   └── employee-attendance-data_cleaning.ipynb                            # Preprocessing & cleaning notebook
├── Data Analysis/
│   ├── employee-attendance-analysis-nabamit.ipynb                         # Full EDA & hypothesis testing notebook
│   └── Bangalore_Attendance_Analytics.ipynb                               # Visualization & KPI scripting notebook
├── Dashboard/
│   └── Screenshot 2026-08-08 204907.png                                   # Looker Studio dashboard mockup/spec
└── Report/
    ├── Project_Development_Guide.md                                       # Developer development guide (This file)
    └── Executive_Analytics_Report.md                                      # Executive findings & statistical report
```

---

## 2. Phase 1: How to Inspect & Verify the Raw Data

Before modifying any data, you must inspect its shape, structure, and quality. This phase ensures you understand the data types, identify missing values, and flag potential anomalies.

### Step 1: Shape and Size Inspection
Load the dataset and check the dimensions to determine the scale of analysis:
```python
import pandas as pd
df = pd.read_csv('Datasets/employee_attendance_bangalore_q1_2026.csv')
print(f"Total Rows: {df.shape[0]} | Total Columns: {df.shape[1]}")
# Expected output: Total Rows: 55374 | Total Columns: 22
```

### Step 2: Columns and Unique Values Summary
Create a summary DataFrame to inspect each column's data type, number of unique values, and a sample value:
```python
info_df = pd.DataFrame({
    "dtype": df.dtypes.astype(str),
    "n_unique": df.nunique(),
    "sample_value": df.iloc[0]
}).reset_index().rename(columns={"index": "column"})
print(info_df)
```

### Step 3: Flag Stray Spaces & Duplicates
Identify trailing/leading whitespace in text columns and locate exact duplicates:
```python
# Check for extra spaces in column headers
bad_headers = [col for col in df.columns if col != col.strip()]
print("Bad headers:", bad_headers)

# Check for extra spaces in text fields
for col in df.select_dtypes(include="object"):
    raw = df[col].astype(str)
    issue_count = (raw != raw.str.strip()).sum()
    if issue_count > 0:
        print(f"Column '{col}' has {issue_count} rows with extra spaces.")

# Check for exact duplicate rows
print(f"Fully duplicated rows: {df.duplicated().sum()}")
```

### Step 4: Missing Value Diagnostic
Determine the missing value counts and percentages across columns:
```python
null_counts = df.isnull().sum()
null_pct = (null_counts / len(df) * 100).round(2)
null_summary = pd.DataFrame({"nulls": null_counts, "pct_missing": null_pct}).loc[null_counts > 0]
print(null_summary)
# Expected output: login_timestamp and logout_timestamp each have 2,635 missing values (4.76%).
```
> [!NOTE]
> Cross-check if the missing timestamps align with employees who are `"On Leave"`:
> `df[df['login_timestamp'].isnull()]['attendance_status'].value_counts()`
> If they match exactly, the missing values are structurally correct (leave days have no log-ins).

---

## 3. Phase 2: How to Clean & Preprocess the Data

In this phase, you resolve all structural issues, strip non-numeric prefixes, convert data types, split complex timestamp fields, and recalculate derived metrics.

### Step A: Strip ID Prefixes & Convert to Integers
Raw IDs are stored as strings (e.g., `ATT0000001`, `VL1000`). Convert them to integers to save memory and improve indexing speed:
```python
df['attendance_id'] = df['attendance_id'].str.replace("ATT", "", regex=False).astype('int64')
df['employee_id'] = df['employee_id'].str.replace("VL", "", regex=False).astype('int64')
```

### Step B: Standardize Text Casing and Trim Spaces
Clean up strings by stripping whitespaces and converting them to title case for consistency:
```python
categorical_cols = ['gender', 'department', 'designation', 'employment_type', 'office_location', 'shift_type', 'attendance_status', 'work_mode', 'leave_type']
for col in categorical_cols:
    df[col] = df[col].astype(str).str.strip().str.title()
```

### Step C: Parse Datetimes and Split Timestamps
Convert raw string columns to datetimes, and split combined date-time timestamps into separate Date and Time columns:
```python
# Convert to proper datetimes
df['login_timestamp'] = pd.to_datetime(df['login_timestamp'], errors='coerce')
df['logout_timestamp'] = pd.to_datetime(df['logout_timestamp'], errors='coerce')
df['attendance_date'] = pd.to_datetime(df['attendance_date'], errors='coerce')
df['date_of_joining'] = pd.to_datetime(df['date_of_joining'], errors='coerce')

# Extract Date and Time components
df['login_date'] = df['login_timestamp'].dt.date
df['login_time'] = df['login_timestamp'].dt.time
df['logout_date'] = df['logout_timestamp'].dt.date
df['logout_time'] = df['logout_timestamp'].dt.time
```

### Step D: Recalculate Time Metrics
Verify and recalculate derived hours to ensure there are no negative numbers, time loops, or discrepancies:
```python
# Recalculate gross hours worked
df['recalculated_gross_hours'] = (df['logout_timestamp'] - df['login_timestamp']).dt.total_seconds() / 3600.0
df['recalculated_gross_hours'] = df['recalculated_gross_hours'].fillna(0.0)

# Recalculate net productive hours (gross hours - break duration converted to hours)
df['recalculated_net_productive_hours'] = df['recalculated_gross_hours'] - (df['break_duration_mins'] / 60.0)
df['recalculated_net_productive_hours'] = df['recalculated_net_productive_hours'].fillna(0.0)

# Enforce non-negative values
df['recalculated_net_productive_hours'] = df['recalculated_net_productive_hours'].clip(lower=0.0)
```

### Step E: Export Cleaned Dataset
Save the cleaned dataset to the `Datasets/` directory as a CSV:
```python
df.to_csv('Datasets/Employee-attendance-and-login-logout-data-bangalore_cleaned.csv', index=False)
```

---

## 4. Phase 3: How to Analyze the Data

Analysis consists of visual exploration (univariate, bivariate, and multivariate analysis) and rigorous statistical testing.

### A. Exploratory Plots
Use `seaborn` and `matplotlib` to analyze distributions:
- **Bivariate Scatter Plot**: Map `total_hours_worked` (X-axis) against `net_productive_hours` (Y-axis) to see correlation.
- **Grouped Box Plots**: Map `net_productive_hours` by `office_location` grouped (`hue`) by `employment_type`.
- **Cross-Tabulation**: Create a normalized stacked bar chart showing the breakdown of `attendance_status` across different `office_location` campuses:
```python
ct = pd.crosstab(df["office_location"], df["attendance_status"], normalize="index") * 100
ct.plot(kind="bar", stacked=True, colormap="Blues_r")
```

### B. Statistical Hypothesis Testing
Implement the following five statistical tests using `scipy.stats` to mathematically prove patterns in workforce behavior:

#### Test 1: Welch's T-Test (Work Mode vs. Productive Hours)
Compare the mean productive hours of office workers (WFO) versus remote workers (WFH):
```python
from scipy import stats
wfo = df[df['work_mode'] == 'Work From Office']['net_productive_hours'].dropna()
wfh = df[df['work_mode'] == 'Work From Home']['net_productive_hours'].dropna()

t_stat, p_val = stats.ttest_ind(wfo, wfh, equal_var=False)
print(f"Welch's t-test: t = {t_stat:.3f}, p = {p_val:.3e}")
```

#### Test 2: Pearson Correlation (Gross Hours vs. Net Productive Hours)
Evaluate the strength of the linear relationship between gross hours worked and active productive hours:
```python
r_val, p_val = stats.pearsonr(df['total_hours_worked'], df['net_productive_hours'])
print(f"Pearson Correlation: r = {r_val:.4f}, p = {p_val:.4e}")
```

#### Test 3: Welch's T-Test (Work Mode vs. Late Arrival Minutes)
Evaluate if remote work reduces punctuality delays (late arrival mins):
```python
group1 = df[df['work_mode'] == 'Work From Office']['late_arrival_mins'].dropna()
group2 = df[df['work_mode'] == 'Work From Home']['late_arrival_mins'].dropna()

t_stat, p_val = stats.ttest_ind(group1, group2, equal_var=False)
print(f"Welch's t-test: t = {t_stat:.3f}, p = {p_val:.4e}")
```

#### Test 4: One-Way ANOVA (Shift Type vs. Productive Hours)
Check if different shift schedules (Flexible, General, Mid, Late, Early) result in significantly different productivity outcomes:
```python
grouped = [group["net_productive_hours"].dropna() for _, group in df.groupby("shift_type")]
f_stat, p_val = stats.f_oneway(*grouped)
print(f"ANOVA: F = {f_stat:.2f}, p = {p_val:.2e}")
```

#### Test 5: Chi-Square Test of Independence (Office Location vs. Attendance Status)
Verify if employee attendance (Present, On Leave, Half Day) is independent of their assigned office location:
```python
contingency = pd.crosstab(df["office_location"], df["attendance_status"])
chi2, chi_p, dof, expected = stats.chi2_contingency(contingency)
print(f"Chi-Square: Chi2 = {chi2:.2f}, p = {chi_p:.4e}")
```

---

## 5. Phase 4: How to Create the Report & Dashboard Spec

Transition the calculated KPIs and findings into the Looker Studio dashboard specification.

### KPI Card Calculations & Verification
Ensure that the metrics displayed on the dashboard match the calculated python statistics exactly:

| Dashboard Card KPI | Python Definition / Code | Value |
| :--- | :--- | :--- |
| **Total Employees** | `df['employee_id'].nunique()` | **980** |
| **Weighted Attendance Rate** | `(Present + 0.5 * Half Day) / Total Records` | **94.17%** |
| **Unweighted Attendance Rate** | `(Present + Half Day) / Total Records` | **95.24%** |
| **Avg Hours Worked** | `df['total_hours_worked'].mean()` | **8.72 hrs** |
| **Avg Productive Hours** | `df['net_productive_hours'].mean()` | **7.73 hrs** |
| **Avg Productivity Ratio** | `Avg Net Productive Hours / Avg Hours Worked` | **88.59%** |
| **Late Arrival Rate** | `(df['late_arrival_mins'] > 0).mean() * 100` | **49.63%** |
| **Early Exit Rate** | `(df['early_exit_mins'] > 0).mean() * 100` | **32.39%** |
| **Avg Overtime Hours** | `df[df['overtime_hours'] > 0]['overtime_hours'].mean()` | **0.63 hrs** |
| **WFH vs WFO Split** | `WFH / (WFH + WFO)` | **21.31%** |
| **Total On Leave** | `(df['attendance_status'] == 'On Leave').sum()` | **2,635** |

> [!TIP]
> Always verify the WFH Share calculation definition. When calculating a WFH-to-WFO split ratio, exclude "Client Site" and "On Leave" records from the denominator to obtain the exact `21.31%` value displayed on the dashboard card. A standard total WFH share calculation (`WFH / Total Records`) yields `19.70%`. Mentioning both numbers in the project documentation prevents confusion between data analytics results and dashboard visual values.
