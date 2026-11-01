# AIRTEL-DATASET-ANALYSIS
Telecom Customer Churn Analysis (SQL + Excel)

An end-to-end analysis of two publicly available Kaggle churn datasets: a 4-operator Indian telecom dataset (Airtel, Reliance Jio, Vodafone, BSNL) and the IBM Telco sample dataset. The work covers data validation, SQL analysis in PostgreSQL, and Excel PivotTables with charts.

Note on the data: both datasets are public Kaggle datasets. They are not official company data, and the telecom dataset shows signs of being generated or anonymised (see Data quality). The project is meant to demonstrate analysis method, not to describe the real performance of any company.

Project goal

Act as a junior analyst given a dataset with no list of questions, and answer:

How big is the customer base, and how is it split (operator, city, age, plan)?
What is the churn rate overall and across segments?
Is any factor associated with higher churn?
Can we trust the data? What needs to be flagged before drawing conclusions?
Datasets
Table	File	Rows x columns	Churn column	Notes
Table 1: Telecom Churn	telecom_churn.csv	243,553 x 14	churn as 0/1	4 operators including Airtel
Table 2: Customer Churn	customer_churn.csv	7,043 x 21	Churn as Yes/No	IBM Telco sample dataset
(original IBM file)	WA_Fn-UseC_-Telco-Customer-Churn.csv	7,043 x 21	Churn as Yes/No	Exact duplicate of customer_churn.csv

Source: Kaggle (add the dataset links and licence here before publishing).

The two tables cannot be joined. Table 1 uses integer IDs (customer_id), Table 2 uses text IDs (customerID), and no ID appears in both. They are analysed separately. The only comparisons between them are side by side on shared categories (gender, overall churn), and they are not evidence about one company versus another.

Table 1 columns

customer_id, telecom_partner, gender, age, state, city, pincode, date_of_registration, num_dependents, estimated_salary, calls_made, sms_sent, data_used, churn

Table 2 columns

customerID, gender, SeniorCitizen, Partner, Dependents, tenure, PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies, Contract, PaperlessBilling, PaymentMethod, MonthlyCharges, TotalCharges, Churn

Repository structure

Adjust folder and file names to match your repository.

.
├── README.md
├── data/
│   ├── telecom_churn.csv
│   ├── customer_churn.csv
│   └── WA_Fn-UseC_-Telco-Customer-Churn.csv
├── sql/
│   ├── 01_setup_and_views.sql
│   ├── 02_telecom_churn_queries.sql
│   ├── 03_customer_churn_queries.sql
│   ├── 04_descriptive_queries.sql
│   └── 05_comparison_queries.sql
├── excel/
│   └── telecom_churn_analysis.xlsx
└── images/
    └── (screenshots of the pivots and charts)
Tools
PostgreSQL 16: validation, views, aggregation, window functions
Microsoft Excel: tables, helper columns, PivotTables, PivotCharts
Workflow
Profile the raw files: row counts, nulls, duplicates, ID uniqueness, value ranges.
Load into PostgreSQL and build views (v_churn, v_customer_churn) that convert text dates, add a Yes/No churn flag, and convert blank TotalCharges to NULL.
Run SQL analysis in two sets, one per table, plus descriptive queries that do not use churn.
Rebuild the key results in Excel with helper columns and PivotTables.
Cross-check: the same numbers must appear in SQL and Excel.
Data quality findings and decisions
#	Finding	Decision
1	Table 1: 19,587 rows (8.0%) have a negative calls_made, sms_sent or data_used value (6,713 / 7,375 / 6,050 respectively). Usage cannot be negative.	Rows are kept. Negative values are excluded from usage averages with FILTER (WHERE x >= 0), and the count is reported.
2	Table 1: every state contains all 6 cities in roughly equal numbers (for example a state paired with a city far from it).	State and city are not combined. Each is used alone, with a caveat.
3	Table 1: date_of_registration is text in dd-mm-yyyy format.	Converted with TO_DATE in the view. pincode is imported as text.
4	Table 1: registration data ends on 4 May 2023, so 2023 is a partial year.	Flagged wherever year-level counts are shown.
5	Table 1: churn is about 20% in almost every group (operator, city, age, salary, gender, usage).	Reported honestly: no strong churn driver appears in this dataset.
6	Table 2: 11 rows have a blank TotalCharges (all with tenure = 0, i.e. brand-new customers).	Imported as text, converted with NULLIF(TRIM(...),'')::NUMERIC.
7	customer_churn.csv is an exact copy of the original IBM file.	Only one copy is analysed.
8	Table 1 stores churn as 0/1, Table 2 as Yes/No.	Table 2 gets a churn_flag (SQL) / churn(0/1) (Excel) helper column so that averages give churn %.
SQL queries

All queries were validated against the data. Each has a business question and an expected result in the SQL files.

Set A: Churn analysis

Table 1 (v_churn)

ID	Question
T1-1	How many customers, and what is the overall churn rate?
T1-2	How many rows have negative usage values?
T1-3	Which operator has the highest churn, ranked with a window function?
T1-4	Does churn differ by city?
T1-5	Does churn differ by age band?
T1-6	Does churn differ by registration year?
T1-7	Does churn differ by salary band?
T1-8	Do operators differ in average usage (negatives excluded)?
T1-9	Do churned and retained customers differ in usage?
T1-10	How does Airtel's churn vary by city?

Table 2 (v_customer_churn)

ID	Question
T2-1	How many customers, and what is the overall churn rate?
T2-2	Is the data clean (unique IDs, blank TotalCharges, tenure 0)?
T2-3	Does churn differ by contract type?
T2-4	Does churn differ by tenure band?
T2-5	Does churn differ by internet service?
T2-6	Does churn differ by payment method?
T2-7	Does churn differ by tech support?
T2-8	Do churned customers pay more and stay for less time?
T2-9	Do senior citizens churn more?
T2-10	How risky is the month-to-month + fiber optic + 12 months or less segment?
Set B: Descriptive queries (no churn column)
N1-1 to N1-10 (Table 1): dataset size, operator share, customers by city and state, age profile, gender split, age and salary by operator, registrations by year, usage profile, usage by gender.
N2-1 to N2-10 (Table 2): contract mix, internet service mix, payment mix, tenure bands, monthly charges summary, revenue share by internet service, contract vs price and tenure, customer demographics, add-on service adoption, contract by internet service.
Set C: Side-by-side comparison (J1 to J5)

Because the tables share no key, comparisons are made by joining two aggregated results on a shared category (gender) or by CROSS JOIN of one-row summaries: dataset size, overall churn rate, gender mix, churn by gender, and the churn gap by gender. These compare two different datasets and do not show that one company is better than another.

Excel analysis

The workbook contains the raw data sheets (telecom_churn, customer_churn), added helper columns, and the PivotTables and charts below.

Helper columns added
Sheet	Column	Purpose
telecom_churn	age_band	18-25, 26-35, 36-45, 46-55, 56+
telecom_churn	salary_band	1. Under 50k, 2. 50k to 100k, 3. 100k+
customer_churn	churn(0/1)	1 if Churn = Yes, else 0, so Average gives churn %
customer_churn	tenure_band	1. 0-12, 2. 13-24, 3. 25-48, 4. 49-72 months

Edit this table to match the columns in your file.

PivotTables

Table 1 (sheet Sheet3)

Pivot	Rows	Values
Customers by operator	telecom_partner	Count of customer_id
Churn rate by city	city	Average of churn
Customers by age band	age_band	Count of customer_id
Churn rate by salary band	salary_band	Average of churn

Table 2 (sheet customer_pivot)

Pivot	Rows	Values
Customers by contract	Contract	Count of customerID
Churn rate by tenure band	tenure_band	Average of churn(0/1)
Customers by internet service	InternetService	Count of customerID
Churn rate by payment method	PaymentMethod	Average of churn(0/1)
Customers by tech support	TechSupport	Count of customerID
Cross-check (SQL = Excel)
Check	Value
Table 1 customers	243,553
Table 1 overall churn	20.05%
Airtel customers	60,905
Table 2 customers	7,043
Table 2 overall churn	26.54%
Key findings

All findings describe associations in these datasets. They do not prove cause.

Table 1: Telecom Churn (4 operators)
Overall churn is 20.05% (48,827 of 243,553 customers).
The four operators have almost the same number of customers (about 25% each). Airtel's churn is 20.37% versus 19.86% to 20.02% for the others, a gap of about 0.5 points that is small and requires further investigation.
Churn is about 20% across cities (19.75% to 20.42%), age bands (19.94% to 20.18%) and salary bands (19.86% to 20.19%).
Average usage of churned and retained customers is about the same.
Takeaway: no strong churn driver appears in this dataset.
Table 2: Customer Churn (IBM Telco sample)
Overall churn is 26.54% (1,869 of 7,043 customers).
Contract type appears strongly associated with churn: month-to-month 42.71%, one year 11.27%, two year 2.83%.
Tenure appears associated with churn: 47.44% in the first 12 months, falling to 9.51% after 48 months.
Fiber optic customers show higher churn (41.89%) than DSL (18.96%).
Customers paying by electronic check show higher churn (45.29%) than those on automatic payments (15.24% to 16.71%).
Customers without tech support show higher churn (41.64%) than those with it (15.17%).
The month-to-month + fiber optic + 12 months or less segment contains 916 customers with a churn rate of 70.20%, versus 20.01% for everyone else.
Churned customers have a higher average monthly charge (74.44 vs 61.27) and a much shorter average tenure (17.98 vs 37.57 months).
Limitations
Both datasets are public Kaggle data. The telecom dataset looks synthetic (uniform ranges, near-equal group sizes, negative usage values), so its findings should not be read as real operator performance.
Correlation is not causation. Statements such as "appears to be associated with" are used on purpose.
The two datasets differ in source, period and definitions, so their churn rates should not be compared as if they were the same measure.
No cost, revenue or customer-satisfaction data is available for Table 1, so no profitability or root-cause analysis is possible.
How to reproduce
Create the tables in PostgreSQL (sql/01_setup_and_views.sql). Import the CSV files with the header option on. Import pincode and totalcharges as text.
Create the views v_churn and v_customer_churn.
Check the counts: SELECT COUNT(*) FROM v_churn; returns 243553 and SELECT COUNT(*) FROM v_customer_churn; returns 7043.
Run the query files and compare each result with the expected values in the comments.
Open the Excel workbook and refresh the PivotTables (Data > Refresh All).
Author

Anushka
