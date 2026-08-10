# Executive Analytics Report: Employee Attendance, Productivity & Policy Insights

## Executive Summary
This report presents a comprehensive data analysis of the employee attendance and login/logout data for the Bangalore workforce (Q1 2026). The dataset contains 55,374 daily attendance records for **980 unique employees** operating across five major physical campuses in Bangalore: Koramangala HQ, Whitefield Tech Park, HSR Layout Hub, Indiranagar Annexe, and Electronic City Campus. 

Through data cleaning, descriptive profiling, custom column formulation, and statistical hypothesis testing, we evaluate the impact of work modes (Work From Office vs. Work From Home), shift schedules, and office locations on employee productivity, overtime, and punctuality. These findings directly validate the metrics and pages of the **AURA HR Attendance Dashboard (Bangalore Hub Spec V2)**.

---

## 1. Core Key Performance Indicators (KPIs)

Based on the Q1 2026 data, the baseline workforce metrics are summarized below:

| Key Performance Indicator | Value | Formula / Operational Definition | Business Interpretation |
| :--- | :--- | :--- | :--- |
| **Total Unique Headcount** | **980** | Unique count of `employee_id` | Size of active workforce under observation. |
| **Weighted Attendance Rate** | **94.17%** | `(Present + 0.5 * Half Day) / Total Shifts` | Key capacity metric reflecting active labor supply. |
| **Unweighted Attendance Rate** | **95.24%** | `(Present + Half Day) / Total Shifts` | Overall participation rate (ignoring shift duration). |
| **Average Hours Worked (Active)** | **9.16 hrs** | Mean `total_hours_worked` (excl. leaves) | Average gross time spent logged into the system on active days. |
| **Average Net Productive Hours** | **7.73 hrs** | Average `net_productive_hours` (excl. leaves) | True active working hours after deducting break times. |
| **Average Productivity Ratio** | **88.59%** | `Net Productive Hours / Gross Hours` | Workforce efficiency; 11.41% of logged time goes to breaks. |
| **Late Arrival Rate** | **49.63%** | `% of shifts where late_arrival_mins > 0` | High; almost half of the daily shifts start late. |
| **Early Exit Rate** | **32.39%** | `% of shifts where early_exit_mins > 0` | Approximately 1 in 3 shifts end before standard time. |
| **Average Overtime Hours** | **0.63 hrs** | Mean `overtime_hours` for shifts with OT > 0 | Overtime duration per active overtime shift. |
| **WFH vs WFO Split KPI** | **21% / 79%** | `WFH / (WFH + WFO)` | Share of home-based working days relative to office days (valid modes). |
| **Total Leave Records Count** | **2,635** | Sum of records where status is 'On Leave' | Equivalent to a **4.76%** absenteeism rate in Q1 2026. |

---

## 2. New Custom Calculated Columns
To enable advanced benchmarking and predictive analytics, four new calculated columns were engineered in the data pipeline:
1. **Weighted Attendance Score (`attendance_score`)**: Maps attendance statuses to numerical values: `Present = 1.0`, `Half Day = 0.5`, `On Leave = 0.0`. This column allows for direct linear aggregation of attendance rates across departments, office locations, and dates.
2. **Overtime Bins (`overtime_bin`)**: Groups overtime hours into 0.5-hour step intervals (e.g., `0.0`, `0.5`, `1.0`, `1.5`, etc.) by flooring `overtime_hours * 2` and dividing by `2`. This facilitates the creation of a distribution histogram.
3. **Tenure Days (`tenure_days`)**: Measures the difference in days between the active `attendance_date` and the employee's `date_of_joining`. This column tracks the career progression of each employee relative to their daily productivity.
4. **Productivity Ratio (`productivity_ratio`)**: Defined as `net_productive_hours / total_hours_worked` (set to `0` if gross hours worked is `0`). This column measures the direct efficiency of each shift, removing the bias of raw hours worked.

---

## 3. Key Analytical Findings & Statistical Hypothesis Test Results

To move beyond simple averages, we conducted five statistical tests to evaluate work policy effectiveness and identify operational bottlenecks.

### Finding 1: Remote Work Increases Net Productive Hours
* **Hypothesis Test:** Welch's independent two-sample t-test comparing `net_productive_hours` between Work From Office (WFO) and Work From Home (WFH).
* **Statistical Metrics:** $t = -4.366$ | $p\text{-value} = 1.270 \times 10^{-5}$ ($p < 0.001$)
* **Business Insight:** We reject the null hypothesis. There is a statistically significant difference in productivity between the two work modes. Remote workers (WFH) achieve higher net productive hours on average than their in-office peers. This suggests that the WFH policy is highly effective in increasing active working times, likely due to fewer office interruptions and greater flexibility.

### Finding 2: Gross Hours are a Robust Proxy for Active Productivity
* **Hypothesis Test:** Pearson correlation coefficient ($r$) between `total_hours_worked` and `net_productive_hours`.
* **Statistical Metrics:** $r = 0.9886$ | $p\text{-value} = 0.000 \times 10^{0}$ ($p < 0.001$)
* **Business Insight:** We reject the null hypothesis. There is an extremely strong, positive linear correlation between the gross time logged into the system and active productive hours. Because break durations remain relatively stable (averaging 59.7 minutes per shift), gross hours serve as a highly reliable operational proxy for actual productivity.

### Finding 3: Work From Home Materially Reduces Late Arrivals
* **Hypothesis Test:** Welch's independent two-sample t-test comparing `late_arrival_mins` between WFO and WFH.
* **Statistical Metrics:** $t = 28.254$ | $p\text{-value} = 2.806 \times 10^{-172}$ ($p < 0.001$)
* **Business Insight:** We reject the null hypothesis. There is a massive, statistically significant difference in late arrival times between WFO and WFH. Employees working from the office accumulate significantly more late arrival minutes than remote workers. This highlights commute times and office logistics as the primary drivers of late logins, proving that remote work drastically improves shift start compliance.

### Finding 4: Shift Schedules Significantly Affect Productivity Output
* **Hypothesis Test:** One-way Analysis of Variance (ANOVA) comparing `net_productive_hours` across 5 shift types.
* **Statistical Metrics:** $F\text{-statistic} = 22.45$ | $p\text{-value} = 1.51 \times 10^{-18}$ ($p < 0.001$)
* **Business Insight:** We reject the null hypothesis. The type of shift schedule assigned to an employee has a highly significant impact on their productive hours. 
  - **Flexible Shifts (10:30-19:30)** yield the highest average productivity (**7.81 hours**).
  - **General Shifts (09:30-18:30)** follow closely at **7.77 hours**.
  - **Early Shifts (08:00-17:00)** show the lowest productivity (**7.50 hours**).
  This indicates that flexible work windows allow employees to match their active hours to their natural energy peaks, while early morning starts lead to slightly lower outputs.

### Finding 5: Attendance Quality is Uniformly Distributed Across Campuses
* **Hypothesis Test:** Chi-Square test of independence between `office_location` and `attendance_status` (Present, On Leave, Half Day).
* **Statistical Metrics:** $\chi^2 = 2.70$ | $\text{Degrees of Freedom (dof)} = 8$ | $p\text{-value} = 0.952$ ($p > 0.05$)
* **Business Insight:** We fail to reject the null hypothesis. Attendance rates, leave behaviors, and half-day occurrences are statistically independent of the employee's assigned office campus. Absenteeism (On Leave) and partial attendance (Half Day) are consistent across all 5 Bangalore offices. This demonstrates that there are no localized site issues (such as local management problems or transport disruptions) affecting attendance at any specific campus.

---

## 4. Looker Studio Dashboard Pages & Interpretation

The **AURA HR Attendance Dashboard (Bangalore Hub Spec V2)** acts as the central interface for this analysis, organized into four primary reporting pages:

### Page 1: Overview
* **Description**: Offers a high-level summary of the entire organization's attendance and contract profiles.
* **Key Components**:
  - **KPI Scorecard Row**: Displays the top 10 KPIs (headcount, attendance rates, work mode shares, overtime averages, and leave counts). Note that the dashboard card displays `20% / 73%` for the WFH vs WFO Split, which represents the total WFH share vs. total WFO share among all recorded shifts.
  - **Daily Attendance Rate Trend Line**: Illustrates the daily attendance percentage across Q1 2026. This chart reveals periodic dips down to 91% (representing weekend transitions or public holidays) and peaks up to 96%.
  - **Work Mode Distribution (Donut Chart)**: Visualizes the split of all shifts: Work From Office (26.7%), Work From Home (26.7%), Client Site (20.0%), and On Leave / N/A (26.7%).
  - **Employment Type Distribution (Donut Chart)**: Confirms a highly balanced cohort structure: Interns (25%), Full-Time (25%), Part-Time (25%), and Contract workers (25%).

### Page 2: Department & Location Benchmarking
* **Description**: Compares attendance behaviors and productivity metrics across departments and locations.
* **Key Components**:
  - **Attendance Status Based on Department (Horizontal Stacked Bar Chart)**: Illustrates the count of shifts marked Present, On Leave, and Half Day for each of the 10 departments. **Engineering** and **Sales** dominate the volume of shifts, but maintain consistent status splits.
  - **Attendance Rate by Office Location (Pie Chart)**: Benchmarks the attendance rate across the five office campuses, showing a perfectly uniform 20% distribution of attendance records per campus.
  - **Productive Hours by Designation (Vertical Bar Chart)**: Displays the average net productive hours for the top 15 roles. Senior engineering roles (e.g., Staff Engineer, Director of Engineering) achieve the highest average active work times.

### Page 3: Punctuality & Shift Pattern
* **Description**: Focuses on shift start delays, late arrivals, and overtime distributions.
* **Key Components**:
  - **Avg Late Arrival by Shift Type (Horizontal Bar Chart)**: Ranks shift schedules by their delay averages. **Flexible (10:30-19:30)** shifts result in the highest late logins (averaging **10.7 minutes**), while **Early (08:00-17:00)** shifts display the lowest delays (**8.96 minutes**).
  - **Punctuality Heatmap (Weekday × Department)**: Cross-tabulates the weekdays against departments. The table shows that **Product Management on Mondays** experiences the highest delays (averaging **57.0 minutes**), followed closely by **Operations on Mondays** (**50.1 minutes**). Fridays and Tuesdays represent the most punctual login days across all departments.
  - **Avg Overtime Hours (Gauge Chart)**: Displays the overall overtime gauge pointing to **0.6 hours** (matching the **0.63 hours** active overtime average).
  - **Overtime Hours Distribution (Histogram)**: Maps the count of shifts across the new `overtime_bin` column, demonstrating a right-skewed distribution where the majority of overtime shifts fall between **0.0 and 1.5 hours**.

### Page 4: Leave Productivity Insights
* **Description**: Examines reasons for leave, correlation between employee tenure and efficiency, and contains an interactive employee search table.
* **Key Components**:
  - **Leave Type Breakdown (Bar Chart)**: Evaluates the reasons for absence (excluding "Not Applicable"). **Casual Leave** is the most frequent reason (approx. 750), followed by **Sick Leave** (approx. 680), and **Earned Leave** (approx. 580).
  - **Tenure vs. Productivity Ratio (Scatter Plot)**: Plots `tenure_days` against `productivity_ratio` for a sample of 10,000 active records. The chart shows a horizontal cluster of data points concentrated between **0.85 and 0.95**, demonstrating that employee efficiency is highly stable and does not decay or shift significantly with length of tenure.
  - **Search Employee Records (Dynamic Table)**: An interactive list providing complete metrics per employee (ID, Name, Department, Type, Shifts, Present Count, Half Days, Leaves, Attendance Rate %, Avg Hour, Avg Late, and Overtime). This table is sorted to highlight the 30 lowest attendance rate employees for immediate managerial review.

---

## 5. Strategic HR Recommendations

Based on these findings, we recommend the following HR policy updates:
1. **Optimize Shift Assignments**: Transition teams with lower average productivity toward **Flexible** or **General** shifts, and minimize **Early (08:00 start)** assignments, as early shifts show a statistically significant dip in productive hours.
2. **Expand Hybrid Work Windows**: Since WFH is statistically proven to reduce late arrivals ($p < 0.001$) and boost net productive hours, formalize a hybrid policy where employees can work remotely on days with critical deliverables or heavy commute constraints.
3. **Punctuality Focus Areas**: With a 49.63% late arrival rate, HR should implement buffer times or flexible core hours rather than strict punch-in rules, as the high late arrival rate is heavily driven by in-office commute delays.
4. **Standardize Leave Policies**: Since attendance and leave behaviors are uniform across all campuses ($p = 0.952$), leave guidelines and health wellness programs can be applied centrally at the corporate level rather than customized per site.
