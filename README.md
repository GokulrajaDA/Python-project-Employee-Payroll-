# Employee Payroll Analytics - Python Project

## 🎯 Data Objective

The primary objective of this project is to analyze **1,599 payroll records of 259 Cook County government employees** from **2016 Q1 to 2017 Q4** to answer key workforce and payroll questions:

1.  **Clean and validate** payroll data (missing offices, zero pay, duplicate employee-period records)
2.  **Track payroll trends** - How does total and average pay change quarter-over-quarter?
3.  **Departmental analysis** - Which bureaus/offices have highest employee count and payroll cost?
4.  **Employee retention** - How long do employees stay and do they have employment gaps?
5.  **Job vs Salary** - Does 1 Job Code = 1 Job Title? What is salary distribution by job?
6.  **Build a normalized relational database** from a single flat file for SQL analysis
7.  **Create a dashboard** for payroll and workforce reporting

This project demonstrates end-to-end data analytics skills: Data Profiling -> Cleaning -> SQL Database Design -> Python Analysis -> Visualization.

## 📊 Dataset Details

**File:** `emp_payroll_project.csv`

| Metric | Value |
|--------|-------|
| **Total Records** | 1,599 |
| **Total Columns** | 15 |
| **Date Range** | 2016Q1 to 2017Q4 (8 quarters) |
| **Unique Employees** | 259 |
| **Unique Position IDs** | 273 |
| **Bureaus** | 16 (Top: Bureau of Health - 697 records) |
| **Offices** | 35 |
| **Job Codes** | 162 |
| **Job Titles** | 161 |

### Columns
- **Time:** Fiscal Year, Fiscal Quarter, Fiscal Period
- **Employee:** First Name, Last Name, Middle Init, Employee Identifier, Original Hire Date
- **Organization:** Bureau, Office, Office Name
- **Job:** Job Code, Job Title, Position ID
- **Payroll:** Base Pay (e.g., $20,088.00)

### Data Quality Issues Found in this file
- Missing Middle Initial: 802 / 1,599 (50% - legitimate NULL)
- Missing Office / Office Name: 3 records
- Base Pay: 0 missing in raw, but 3 missing after cleaning $ and , | 20 zero-pay records
- Duplicate Employee + Fiscal Period: 43 combinations (same employee appears twice in same quarter)
- 162 Job Codes vs 161 Job Titles - needs mapping validation

## 🔧 Python Work - What to Do

### 1. Data Cleaning (First Step)
```python
import pandas as pd

df = pd.read_csv("emp_payroll_project.csv")

# Clean Base Pay: $20,088.00 -> 20088.00 float
df["Base Pay Clean"] = pd.to_numeric(
    df["Base Pay"].astype(str)
      .str.replace("$","", regex=False)
      .str.replace(",","", regex=False),
    errors="coerce"
)

# Clean Hire Date
df["Original Hire Date"] = pd.to_datetime(df["Original Hire Date"], errors="coerce")

print(f"Missing after clean: {df['Base Pay Clean'].isna().sum()}")

2. Key Analytics Questions 
A. Payroll Trend
python
trend = df.groupby("Fiscal Period")["Base Pay Clean"].sum()
# Is total payroll increasing from 2016Q1 (207 records) to 2017Q4 (198 records)?
B. Bureau Analysis
python
bureau = df.groupby("Bureau").agg(
    employees=('Employee Identifier','nunique'),
    total_pay=('Base Pay Clean','sum')
).sort_values('total_pay', ascending=False)
C. Duplicate Investigation (43 cases)
python
# Why does same employee have 2 records in same quarter?
dup = df[df.duplicated(['Employee Identifier','Fiscal Period'], keep=False)]
# Hypothesis: Multiple positions / payroll adjustment
D. Job Code Consistency
python
# Check if 1 Job Code = 1 Title
df.groupby("Job Code")["Job Title"].nunique().value_counts()

🗄️ Proposed Normalized Database
For SQL project, split flat file into 5 tables:
1. employees (259 rows)
Employee_ID (PK), First_Name, Last_Name, Middle_Init, Hire_Date
2. departments (35 rows)
Dept_ID (PK), Bureau, Office_Code, Office_Name
3. jobs (162 rows)
Job_ID (PK), Job_Code, Job_Title
4. positions (273 rows)
Position_ID (PK), Employee_ID (FK), Job_ID (FK), Dept_ID (FK)
5. payroll (1,599 rows)
Payroll_ID (PK), Position_ID (FK), Fiscal_Period, Base_Pay
Benefits: Removes repetition, enforces referential integrity, easier queries.

📈 Expected Outputs / Visualizations
Line chart: Total payroll by 8 fiscal periods
Bar chart: Top 10 Bureaus by employee count and total pay
Boxplot: Salary distribution per Bureau
Heatmap: Missing value analysis
Table: Top 5 highest paid Job Titles
Employee tenure: Quarters worked per employee (1 to 8 quarters)
print(f"Zero pay: {(df['Base Pay Clean']==0).sum()}")
print(f"Avg pay: ${df['Base Pay Clean'].mean():.2f}")

⚠️ Limitations & Notes
File covers only 2016-2017 (8 quarters), not full 2016-2018 history
Zero pay (20 records) may be legitimate adjustments, not errors - investigate
3 missing office records need to be matched with official Cook County office list
Do not delete 43 duplicate employee-period records without business rule verification

🎯 Final Deliverables for GitHub
Data profiling report
Cleaned dataset
Normalized SQL database (5 tables)
6+ Visualizations
Data quality dashboard
Documentation of limitations[x]
