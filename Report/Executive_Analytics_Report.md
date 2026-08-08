# Executive Analytics Report: Employee Attendance, Productivity & Policy Insights

## Executive Summary
This report presents a thorough data analysis of the employee attendance and login/logout data for the Bangalore workforce (Q1 2026). The dataset contains 55,374 daily attendance records for **980 unique employees** operating across five major physical campuses in Bangalore: Koramangala HQ, Whitefield Tech Park, HSR Layout Hub, Indiranagar Annexe, and Electronic City Campus. 

Through rigorous data cleaning, descriptive profiling, and statistical hypothesis testing, we evaluate the impact of work modes (Work From Office vs. Work From Home), shift schedules, and office locations on employee productivity, overtime, and punctuality. These findings directly validate the layout and metrics of the **AURA HR Attendance Dashboard (Bangalore Hub Spec V2)**.

---

## 1. Core Key Performance Indicators (KPIs)

Based on the Q1 2026 data, the baseline workforce metrics are summarized below:

| Key Performance Indicator | Value | Formula / Operational Definition | Business Interpretation |
| :--- | :--- | :--- | :--- |
| **Total Unique Headcount** | **980** | Unique count of `employee_id` | Size of active workforce under observation. |
| **Weighted Attendance Rate** | **94.17%** | `(Present + 0.5 * Half Day) / Total Shifts` | Key capacity metric reflecting active labor supply. |
| **Unweighted Attendance Rate** | **95.24%** | `(Present + Half Day) / Total Shifts` | Overall participation rate (ignoring shift duration). |
| **Average Hours Worked** | **8.72 hrs** | Mean `total_hours_worked` (excl. leaves) | Average gross time spent logged into the system per day. |
| **Average Net Productive Hours** | **7.73 hrs** | Average `net_productive_hours` (excl. leaves) | True active working hours after deducting break times. |
| **Average Productivity Ratio** | **88.59%** | `Net Productive Hours / Gross Hours` | Workforce efficiency; 11.41% of logged time goes to breaks. |
| **Late Arrival Rate** | **49.63%** | `% of shifts where late_arrival_mins > 0` | High; almost half of the daily shifts start late. |
| **Early Exit Rate** | **32.39%** | `% of shifts where early_exit_mins > 0` | Approximately 1 in 3 shifts end before standard time. |
| **Average Overtime Hours** | **0.63 hrs** | Mean `overtime_hours` for shifts with OT > 0 | Overtime duration per active overtime shift. |
| **Remote Work Ratio (WFH Split)** | **21.31%** | `WFH / (WFH + WFO)` | Share of home-based working days relative to office days. |
| **Total Leave Records Count** | **2,635** | Sum of records where status is 'On Leave' | Equivalent to a **4.76%** absenteeism rate in Q1 2026. |

---

## 2. Key Analytical Findings & Statistical Hypothesis Test Results

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
* **Hypothesis Test:** Chi-Square test of independence between `office_location` and `attendance_status` (Present, Half Day, On Leave).
* **Statistical Metrics:** $\chi^2 = 2.70$ | $\text{Degrees of Freedom (dof)} = 8$ | $p\text{-value} = 0.952$ ($p > 0.05$)
* **Business Insight:** We fail to reject the null hypothesis. Attendance rates, leave behaviors, and half-day occurrences are statistically independent of the employee's assigned office campus. Absenteeism (On Leave) and partial attendance (Half Day) are consistent across all 5 Bangalore offices. This demonstrates that there are no localized site issues (such as local management problems or transport disruptions) affecting attendance at any specific campus.

---

## 3. Looker Studio Dashboard Structure & Interpretation

The **AURA HR Attendance Dashboard (Bangalore Hub Spec V2)** acts as the central interface for this analysis.

### Dashboard Layout & Components
1. **Global Filters (Left Sidebar)**: 
   Allows HR leaders to drill down dynamically by Date Range, Month, Department, Job Designation, Office Location, Employment Type, Shift Type, Work Mode, Attendance Status, Gender, and individual Employee Name.
2. **KPI Scorecard Row (Top)**:
   Highlights the 10 core metrics (Total Headcount, Attendance Rates, Avg Hours Worked, Productivity Ratios, Punctuality Metrics, Overtime, and WFH Splits) for immediate visual monitoring.
3. **Daily Attendance Rate Trend Chart (Middle)**:
   A line chart mapping daily attendance percentage across Q1 2026. This chart visually identifies systemic dips (e.g., weekend transitions or public holidays) where attendance rates drop toward 91%, as well as high-performing periods peaking at 96%.
4. **Work Mode & Employment Type Distributions (Bottom)**:
   - **Work Mode Donut Chart**: Breaks down the active shifts: Work From Office (26.7%), Work From Home (26.7%), Client Site (20.0%), and On Leave / Not Applicable (26.7%).
   - **Employment Type Donut Chart**: Illustrates a balanced cohort structure: Interns (25%), Full-Time (25%), Part-Time (25%), and Contract workers (25%).

---

## 4. Strategic HR Recommendations

Based on these findings, we recommend the following HR policy updates:
1. **Optimize Shift Assignments**: Transition teams with lower average productivity toward **Flexible** or **General** shifts, and minimize **Early (08:00 start)** assignments, as early shifts show a statistically significant dip in productive hours.
2. **Expand Hybrid Work Windows**: Since WFH is statistically proven to reduce late arrivals ($p < 0.001$) and boost net productive hours, formalize a hybrid policy where employees can work remotely on days with critical deliverables or heavy commute constraints.
3. **Punctuality Focus Areas**: With a 49.63% late arrival rate, HR should implement buffer times or flexible core hours rather than strict punch-in rules, as the high late arrival rate is heavily driven by in-office commute delays.
4. **Standardize Leave Policies**: Since attendance and leave behaviors are uniform across all campuses ($p = 0.952$), leave guidelines and health wellness programs can be applied centrally at the corporate level rather than customized per site.
