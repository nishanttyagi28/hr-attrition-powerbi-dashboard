# HR Attrition Dashboard: Power BI

![HR Attrition Dashboard demo](assets/hr-attrition-demo.gif)

An interactive **Power BI** dashboard that answers one HR question: **who is leaving the company, and why?**
It's built on the IBM HR Analytics Employee Attrition dataset (1,470 employees, 35 attributes) and saved as a **Power BI Project (`.pbip`)**, so every query, DAX measure and visual is plain text that you can diff and review in Git.

▶️ Full-quality demo video: [`assets/hr-attrition-demo.mp4`](assets/hr-attrition-demo.mp4)

---

## Problem statement

Employee attrition is expensive. Each exit costs hiring, onboarding, lost productivity and team knowledge.
HR leaders need to know:

1. **How big is the problem?** Headcount, number of leavers, attrition rate.
2. **Where is it concentrated?** By department, job role, age, gender and satisfaction.
3. **What drives it?** Overtime, pay, tenure, travel, marital status and work-life balance.

This dashboard puts those answers on two report pages, and every page can be sliced by **Department, Gender, Age Band and OverTime**.

## Key insights

All figures come from the dataset as shown in the dashboard (and were cross-checked in pandas).

| Metric | Value |
|---|---|
| Headcount | **1,470** |
| Attrited employees | **237** |
| Attrition rate | **16.1%** |
| Active employees | 1,233 |
| Avg monthly income | $6,503 |

- **Overtime is the biggest single driver.** Employees who work overtime leave at **30.5%**, about 3× the **10.4%** rate of those who don't (127 of 416 vs 110 of 1,054).
- **Leavers earn much less.** Average monthly income is **$4,787 for leavers vs $6,833 for those who stay**. The lowest income band (< $3K/month) has **28.6%** attrition, compared with **3.8%** for $15K+.
- **Early tenure is the riskiest period.** **34.9%** of employees with 0–1 years at the company leave, falling to 9.7% at 20+ years.
- **Young employees churn the most.** The 18–24 age band has **39.2%** attrition, against 10.1% for ages 35–44.
- **Sales has the highest department attrition (20.6%)**, followed by HR (19.0%) and R&D (13.8%). By role, Laboratory Technicians (62) and Sales Executives (57) account for the most leavers.
- **Frequent travellers leave at 24.9%**, compared with 15.0% for rare travellers and 8.0% for non-travellers.
- **Single employees leave at 25.5%**, compared with 12.5% for married and 10.1% for divorced employees.
- **Low job satisfaction matters.** Attrition is 22.8% at satisfaction level 1 (Low) and 11.3% at level 4 (Very High). For work-life balance, "Bad" (1) is ~31%.
- **Hotspot example (shown in the demo):** Sales employees who also work overtime: 128 people, 48 leavers, a **37.5%** attrition rate.

## Report pages and visuals

### 1. Overview
![Overview page](assets/overview.png)

- **Slicers (dropdown):** Department, Gender, Age Band, OverTime
- **KPI cards:** Headcount, Attrited Employees, Attrition Rate, Active Employees, Avg Monthly Income, Avg Age
- **Attrition Rate by Department** (bar)
- **Attrited Employees by Job Role** (bar)
- **Attrition Rate by Age Band** (column)
- **Attrited Employees: OverTime vs Not** (donut)
- **Attrition Rate by Gender** (column)
- **Attrition Rate by Job Satisfaction** (column)

### 2. Attrition Drivers
![Drivers page](assets/drivers.png)

- **Same four slicers** as the Overview page
- **KPI cards:** OverTime Attrition Rate, Non-OverTime Attrition Rate, Avg Income (Attrited), Avg Income (Active), Avg Years at Company, Avg Job Satisfaction (1–4)
- **Attrition Rate by Monthly Income Band** (< 3K, 3K–6K, 6K–10K, 10K–15K, 15K+)
- **Attrition Rate by Tenure** (0–1, 2–4, 5–9, 10–19, 20+ years)
- **Attrition Rate by Business Travel**
- **Attrition Rate by Marital Status**
- **Attrition Rate by Work-Life Balance**
- **Attrition Rate by Education Field**

The report uses a custom teal theme (`HRTealTheme.json`). Leaver-related KPIs are highlighted in red.

## Data model and Power Query (M) steps

There is a single import table, **`HR`**, loaded from `data/WA_Fn-UseC_-HR-Employee-Attrition.csv`:

1. **Source:** `Csv.Document(File.Contents(...))`, UTF-8, 35 columns
2. **Promote headers**
3. **Change types:** 8 text columns and the rest as whole numbers (`en-US` culture)
4. **Remove constant columns:** `EmployeeCount`, `Over18`, `StandardHours` (the same value for every row)
5. **`AttritionFlag`:** 1 if `Attrition = "Yes"`, otherwise 0 (makes the measures simple sums)
6. **`Age Band`** (18-24 … 55+) and **`Income Band`** (< 3K … 15K+), each with a hidden sort column so the axes sort in logical order
7. **`Tenure Band`** from `YearsAtCompany` (0-1 yrs … 20+ yrs), with a sort column
8. **Readable labels** for the 1–4 survey scores: `Job Satisfaction Level`, `Environment Satisfaction Level`, `Work-Life Balance Level`
9. **`Business Travel`** relabelled (Travels Rarely / Travels Frequently / Non-Travel)

## DAX measures

13 measures sit in the `KPIs` display folder:

| Measure | Purpose |
|---|---|
| `Total Employees` | Headcount |
| `Attrited Employees` | Number of leavers |
| `Active Employees` | Headcount minus leavers |
| `Attrition Rate %` | Leavers ÷ headcount |
| `Retention Rate %` | 1 − attrition rate |
| `Avg Monthly Income` | Average pay |
| `Avg Age` | Average age |
| `Avg Years at Company` | Average tenure |
| `Overtime Attrition Rate` | Attrition rate where OverTime = Yes |
| `Non-Overtime Attrition Rate` | Attrition rate where OverTime = No |
| `Avg Job Satisfaction` | Average 1–4 score |
| `Avg Income - Attrited` | Average pay of leavers |
| `Avg Income - Active` | Average pay of current employees |

A few of the formulas:

```DAX
Total Employees      = COUNTROWS ( HR )
Attrited Employees   = SUM ( HR[AttritionFlag] )
Attrition Rate %     = DIVIDE ( [Attrited Employees], [Total Employees] )

Overtime Attrition Rate =
    CALCULATE ( [Attrition Rate %], HR[OverTime] = "Yes" )

Avg Income - Attrited =
    CALCULATE ( [Avg Monthly Income], HR[Attrition] = "Yes" )
```

## How to open

**Requirements:** Power BI Desktop on Windows (free). Built with version 2.157 (Microsoft Store).

1. Clone or download this repo.
2. Double-click **`HR Attrition Dashboard.pbip`**.
3. **Update the CSV path.** The query uses an absolute path from the author's machine:
   `C:\Users\Admin\Documents\HR-Attrition-Dashboard\data\WA_Fn-UseC_-HR-Employee-Attrition.csv`
   If your copy is in a different folder: **Transform data → select `HR` → Advanced Editor**, change the path in the `Source` step to your local `data\WA_Fn-UseC_-HR-Employee-Attrition.csv`, then **Close & Apply**.
   (Alternatively, use **Transform data → Data source settings → Change Source…**.)
4. Click **Refresh** on the Home ribbon.
5. If Power BI asks to upgrade the project format, either choice works. Choosing **Don't upgrade** keeps the files as committed.

Want a single shareable file? Use **File → Save as → `.pbix`**.

> The data cache (`.pbi/cache.abf`) and per-user settings (`.pbi/localSettings.json`) are deliberately not committed (see `.gitignore`), so the first open needs a **Refresh** to load the data.

## Repository structure

```
├── HR Attrition Dashboard.pbip              # open this
├── HR Attrition Dashboard.Report/           # report layout, visuals, theme
│   ├── report.json
│   ├── definition.pbir
│   └── StaticResources/RegisteredResources/HRTealTheme.json
├── HR Attrition Dashboard.SemanticModel/    # data model: Power Query M + DAX (model.bim)
├── data/WA_Fn-UseC_-HR-Employee-Attrition.csv
└── assets/                                  # demo GIF/MP4 and screenshots
```

## Tools

Power BI Desktop · Power Query (M) · DAX · Power BI Project format (PBIP) · Git/GitHub

## Data source and license

- **Dataset:** IBM HR Analytics Employee Attrition & Performance. It is a **fictional** dataset created by IBM data scientists.
- **File obtained from:** [IBM/employee-attrition-aif360](https://github.com/IBM/employee-attrition-aif360) (`data/emp_attrition.csv`, byte-identical to the Kaggle file `WA_Fn-UseC_-HR-Employee-Attrition.csv`). Originally published on [Kaggle](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset).
- **Data license**, as stated by the IBM repo: the database is under the [Open Database License (ODbL) v1.0](https://opendatacommons.org/licenses/odbl/1-0/) and its contents are under the [Database Contents License (DbCL) v1.0](https://opendatacommons.org/licenses/dbcl/1-0/). The CSV is redistributed here unmodified, with this attribution. (The IBM repository's code is Apache-2.0.)
- Credit to IBM and the original dataset authors. This project is not affiliated with or endorsed by IBM.

## Author

**Nishant Tyagi**
[LinkedIn](https://www.linkedin.com/in/nishant-tyagi-7aa458356/) · [GitHub](https://github.com/nishanttyagi28)

If you found this useful, a ⭐ is appreciated. Feedback and suggestions are welcome via Issues.
