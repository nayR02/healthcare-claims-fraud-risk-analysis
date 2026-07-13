# healthcare-claims-fraud-risk-analysis
Power BI dashboard analyzing fraud risk patterns in healthcare claims


A two-page Power BI dashboard identifying fraud risk patterns in healthcare claims, built on a cleaned and standardized Kaggle dataset. Combines an executive summary view with a detailed drill-down matrix for risk investigation.

Surfaces fraud rate trends over time, demographic and geographic fraud concentration (age group, gender, state), and a custom Risk Index to rank provider specialties and states by fraud exposure.

Data cleaning/standardization, DAX measure design, fraud/risk analytics, multi-page dashboard UX (overview + drill-down), geographic and demographic segmentation

Source: Kaggle (cleaned/standardized by me)**this data is synthetic


Page 1 – Dashboard Overview:

KPI cards: Claim Count, Fraud Rate, Total Claim Amount, Fraudulent Claim Amount
Ribbon chart: Fraud Risk by Age Group
100% stacked bar: Fraud Cases by State (fraudulent vs. not)
Line chart: Fraud Trend Over Time (count + fraud rate)
Donut: Claims by Gender
Funnel: Claim Volume by Age Group
Filled map: Fraud rate by state
Slicers: Date, City, Provider Specialty, Provider Type

Page 2 – Fraud Risk Summary (Matrix):

Pivot table: State × Provider Specialty × Age Group, with Fraudulent Claim Amount, Avg Fraudulent Claim Amount, and a custom Risk Index measure
Card: Top 3 Risk States (another calculated measure)
Slicers: Admission Type, Provider Specialty


(one of my first BI projects)
