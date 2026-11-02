Telecom Churn Analysis (SQL, Excel, Power BI)

An end-to-end analysis of two publicly available Kaggle churn datasets: a 4-operator Indian telecom dataset (Airtel, Reliance Jio, Vodafone, BSNL, with an Airtel focus) and the IBM Telco sample dataset. The project covers data validation and analysis in PostgreSQL, PivotTables and charts in Excel, and an interactive Power BI dashboard.

Note on the data: both datasets are public Kaggle datasets. They are not official company data, and the telecom dataset shows signs of being generated or anonymised (see Data quality). The project demonstrates analysis method, not the real performance of any company.

Project goal

Act as a junior analyst given a dataset with no list of questions, and answer:

How big is the customer base, and how is it split (operator, city, age, plan)?
What is the churn rate overall and across segments?
Is any factor associated with higher churn?
Can the data be trusted? What must be flagged before drawing conclusions?
Datasets
Table	File	Rows x columns	Churn column	Notes
Table 1: Telecom Churn	telecom_churn.csv	243,553 x 14	churn as 0/1	4 operators including Airtel
Table 2: Customer Churn	customer_churn.csv	7,043 x 21	Churn as Yes/No	IBM Telco sample dataset
(original IBM file)	WA_Fn-UseC_-Telco-Customer-Churn.csv	7,043 x 21	Churn as Yes/No	Exact duplicate of customer_churn.csv

Source: Kaggle (add the dataset links and licence here before publishing).

The two tables cannot be joined. Table 1 uses integer IDs (customer_id), Table 2 uses text IDs (customerID), and no ID appears in both. They are analysed separately, in separate SQL views, separate pivots and separate Power BI tables with no relationships. Comparisons between them are only side by side on shared categories (gender, overall churn) and are not evidence about one company versus another.

Airtel: only Table 1 contains Airtel (60,905 of 243,553 customers, one of four operators). Table 2 has no operator column and is not Airtel data.

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
├── powerbi/
│   └── Telecom_Churn_Dashboard.pbix
└── images/
    └── (screenshots of the pivots, charts and dashboard)
Tools
PostgreSQL 16: validation, views, aggregation, window functions
Microsoft Excel: tables, helper columns, PivotTables, PivotCharts
Power BI Desktop: Power Query cleaning, DAX measures, interactive dashboard
Workflow
Profile the raw files: row counts, nulls, duplicates, ID uniqueness, value ranges.
Load into PostgreSQL and build views (v_churn, v_customer_churn) that convert text dates, add a Yes/No churn flag, and convert blank TotalCharges to NULL.
Run SQL analysis in sets: churn analysis per table, descriptive queries that do not use churn, and side-by-side comparisons.
Rebuild the key results in Excel with helper columns and PivotTables.
Build the Power BI dashboard with DAX measures and interactive visuals.
Cross-check: the same key numbers must appear in SQL, Excel and Power BI.
Data quality findings and decisions
#	Finding	Decision
1	Table 1: 19,587 rows (8.0%) have a negative calls_made, sms_sent or data_used value (6,713 / 7,375 / 6,050 respectively). Usage cannot be negative.	Rows are kept. Negative values are excluded from usage averages (FILTER (WHERE x >= 0) in SQL, data_used >= 0 in DAX), and the count is reported.
2	Table 1: every state contains all 6 cities in roughly equal numbers (for example a state paired with a city far from it).	State and city are never combined. Only city is used (map and slicer), with a caveat.
3	Table 1: date_of_registration is text in dd-mm-yyyy format.	Converted with TO_DATE in SQL, and with Change Type > Using Locale (English, United Kingdom) in Power Query. pincode is imported as text.
4	Table 1: registrations end on 4 May 2023, so May 2023 (600 customers) is a partial month.	Excluded from the monthly trend line. Year-level counts for 2023 are flagged as partial.
5	Table 1: churn is about 20% in almost every group (operator, city, age, salary, gender, usage).	Reported honestly: no strong churn driver appears in this dataset.
6	Table 2: 11 rows have a blank TotalCharges (all with tenure = 0, brand-new customers).	Imported as text and converted in SQL (NULLIF(TRIM(...),'')::NUMERIC). In Power Query: Trim, change to Decimal, Replace Errors with null.
7	customer_churn.csv is an exact copy of the original IBM file.	Only one copy is analysed.
8	Table 1 stores churn as 0/1, Table 2 as Yes/No.	Table 2 gets a churn_flag (SQL) and churn(0/1) (Excel) helper column. DAX uses churn = 1 for Table 1 and Churn = "Yes" for Table 2.
9	Power BI: grouping by calendar month name mixes years (Jan to Apr include 4 years, May to Dec only 3), which creates a false seasonal drop.	The trend uses Month Start as a date (Jan-2020 to Apr-2023, 40 months).
10	Power BI: columns from one table (for example Contract, InternetService, risk_segment) cannot split or filter the other table's measures. Doing so repeats the overall total.	Each visual uses fields from one table only.
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

Because the tables share no key, comparisons join two aggregated results on a shared category (gender), or CROSS JOIN one-row summaries: dataset size, overall churn rate, gender mix, churn by gender, and the churn gap by gender. These compare two different datasets and do not show that one company is better than another.

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
Power BI dashboard

The dashboard is built on the telecom table with no relationships (a single table). Cleaning is done in Power Query, and every number is calculated with DAX measures.

Power Query steps
pincode set to Text
date_of_registration converted with Using Locale (English, United Kingdom) and renamed reg_date
Numeric columns set to Whole Number
Negative usage values left unchanged and handled in the measures
Calculated columns
Column	Purpose
age_band	18-25, 26-35, 36-45, 46-55, 56+
Month Start	First day of the registration month, used for the trend line
Map Location	City plus ", India", for the map (city only, never state)
DAX measures
Measure	Definition	Value (no slicers)
Telecom Customers	COUNTROWS(telecom_churn)	243,553
Telecom Churned	CALCULATE(COUNTROWS(telecom_churn), telecom_churn[churn] = 1)	48,827
Telecom Churn %	DIVIDE([Telecom Churned], [Telecom Customers])	20.05%
Retention %	1 - [Telecom Churn %]	79.95%
Airtel Churn %	CALCULATE([Telecom Churn %], telecom_churn[telecom_partner] = "Airtel")	20.37%
Airtel Customers	CALCULATE([Telecom Customers], telecom_churn[telecom_partner] = "Airtel")	60,905

Airtel Churn % always shows Airtel, even if the operator slicer is set to another operator, because its filter overrides the slicer. It still responds to the city slicer.

Visuals
Type	Fields	What it shows
KPI cards	Telecom Customers, Telecom Churn %, Airtel Churn % (and Airtel Customers)	Headline numbers
Slicers	telecom_partner, city (optional: gender, age_band)	Interactive filtering
Bar chart	Y-axis: telecom_partner, X-axis: Telecom Churn %	Churn by operator (axis starts at 0)
Column chart	X-axis: age_band, Y-axis: Telecom Customers	Customer profile by age band
Line chart	X-axis: Month Start, Y-axis: Telecom Customers	Registrations from Jan-2020 to Apr-2023 (May 2023 excluded)
Pie or donut	Legend: telecom_partner, Values: Telecom Customers	Customer share by operator
Map	Location: Map Location, Bubble size: Telecom Customers	Customers by city
Table	telecom_partner, Telecom Customers, Telecom Churned, Telecom Churn %	Operator scorecard

Update this table to match the final dashboard. If you also built a dashboard on the IBM table, add it here with its own measures (IBM Customers, IBM Churn %, High-risk Churn %) and visuals.

Dashboard notes
Each visual uses fields from one table only.
Pies and donuts show customer counts, never churn percentages or averages.
Churn charts start their axis at 0, because the differences between operators are under 1 point.
Dashboard text states the data-quality caveat (19,587 rows with negative usage) and that the source is a public Kaggle dataset.
Cross-check (SQL = Excel = Power BI)
Check	Value
Table 1 customers	243,553
Table 1 churned customers	48,827
Table 1 overall churn	20.05%
Airtel customers	60,905
Airtel churn	20.37%
Table 2 customers	7,043
Table 2 overall churn	26.54%
Key findings

All findings describe associations in these datasets. They do not prove cause.

Table 1: Telecom Churn (4 operators)
Overall churn is 20.05% (48,827 of 243,553 customers).
The four operators have almost the same number of customers (about 25% each). Airtel's churn is 20.37% versus 19.86% to 20.02% for the others (19.94% combined), a gap of about 0.4 points that is small and requires further investigation.
Churn is about 20% across cities (19.75% to 20.42%), age bands (19.94% to 20.18%) and salary bands (19.86% to 20.19%).
Average usage of churned and retained customers is about the same.
Airtel's highest-churn city is Hyderabad (21.36%), but with about 10,000 customers per city the difference from other cities may be noise.
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
