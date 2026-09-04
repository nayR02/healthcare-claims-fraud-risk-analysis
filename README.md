# Healthcare Claims Fraud Risk Analysis

A Power BI fraud-risk dashboard built on a synthetic 20,100-row healthcare claims dataset. Combines an executive overview with a drill-down risk matrix, ranking states, provider specialties, and age groups by fraud exposure using a custom Risk Index — and surfaces a real data-quality problem in the source data along the way.

---

## Overview

| | |
|---|---|
| **Dataset** | 20,100 synthetic healthcare claims (Kaggle, cleaned/standardized, staged in SQL Server) |
| **Tool** | Power BI Desktop (DAX, SQL Server source via Power Query) |
| **Report pages** | Dashboard Overview · Fraud Risk Summary (Matrix) |
| **Total claim amount** | $25.03B *(synthetic scale)* |
| **Fraudulent claim amount** | $6.51B |
| **Overall fraud rate** | 24.93% |
| **Potential loss ratio** | 26.02% |
| **Avg fraudulent claim amount** | $1.30M |
| **Top 3 risk states (by fraudulent $)** | WI, NY, NC |

This was one of my first BI projects — the DAX and dashboard structure hold up, but going back through it with a fraud-analyst lens surfaced a real issue I missed the first time. Documented below instead of quietly fixed and hidden.

---

**Dashboard Overview** — KPI cards (claim count, fraud rate, claim amounts), Fraud Risk by Age Group ribbon chart, Fraud Cases by State 100% stacked bar, Fraud Trend Over Time, Claims by Gender donut, Claim Volume by Age Group funnel, fraud rate by state map. Slicers: Date, City, Provider Specialty, Provider Type.

**Fraud Risk Summary (Matrix)** — State × Provider Specialty × Age Group pivot with Fraudulent Claim Amount, Avg Fraudulent Claim Amount, and Risk Index. Top 3 Risk States card. Slicers: Admission Type, Provider Specialty.

---

## Data Model

A single flat fact table (`SynHealthClaims`, 20,100 rows × 37 columns), sourced from a local SQL Server database rather than a raw CSV — the cleaned Kaggle data was staged in SQL Server (`HealthClaimsDB`) and pulled into Power BI via `Sql.Database()` in Power Query.

Unlike my later projects (Corporate Expense Audit, FinTech Analytics), there's no explicit star schema here — no separate Dates dimension table joined to the fact table. Power BI auto-generated hidden local date tables in the background instead. That works fine at this scale for the charts this dashboard needed, but it's a conscious gap compared to the relational modeling in my later work — I'd build an explicit calendar table if this project needed real time-intelligence measures (`DATEADD`, `TOTALYTD`, etc.).

**Key calculated columns:**
- `AgeGroup`, `Distance_Category`, `Patient_Claim_History` — bucketing via `SWITCH(TRUE(), ...)`
- `Fraud_Label` — readable Fraud/Not Fraud flag for visuals
- `Gender Reporting` — normalizes blank/"Other" gender values to "Not Specified"
- `Unique_Claim_Key` — `Claim_ID & "-" & Patient_ID`, intended as a uniqueness check (see talking points — it doesn't actually catch the duplicates below)

---

## DAX Measures

| Measure | Purpose |
|---|---|
| `Fraud_RateM` | Overall fraud rate — fraudulent claims ÷ total claims |
| `Fraudulent_Claim_Amount` | Sum of claim amounts where `Is_Fraudulent = TRUE` |
| `Avg_Fraudulent_Claim_Amount` | Average fraudulent claim size |
| `Potential_Loss_Ratio` | Fraudulent $ ÷ Total $ |
| `Risk_Index` | Weighted composite risk score *(see talking points — currently a duplicate, not a real composite)* |
| `Fraud_Rank` | Ranks Patient_State by fraud rate |
| `Fraud_Rank_State` | Ranks Patient_State by fraudulent claim amount |
| `Top3_Fraud_Rate` | Isolates the top-3-ranked fraud rates |
| `Top3_Risk_States` | Concatenated string of the top 3 states by fraudulent $ |

---

## Technical Decisions & Talking Points

Honest issues found on revisiting this project, not just a feature list.

**A duplicate claim can be flagged fraudulent on one row and clean on its twin**
`Claim_ID` has 20,000 unique values across 20,100 rows — 100 exact-duplicate pairs (same patient, same claim date, same amount). In several pairs, the two rows disagree on `Is_Fraudulent` — the same claim is simultaneously flagged and not flagged, depending on which duplicate row a visual happens to aggregate. The `Unique_Claim_Key` column (`Claim_ID & "-" & Patient_ID`) looks like it was meant to catch this, but since Claim_ID and Patient_ID are identical between the duplicate rows, the key doesn't actually distinguish them — it's a check that doesn't check anything. **What I'd do differently:** add an explicit duplicate-detection measure (matching on Claim_ID + Patient_ID + Claim_Date + Claim_Amount, the way Corporate Expense Audit's duplicate rule works) and either de-duplicate at the Power Query stage or surface the conflict as its own flag for a reviewer to resolve, rather than letting it silently skew the fraud rate in whichever direction the last-loaded row happens to point.

**Risk_Index isn't actually a composite score**
```dax
Risk_Index = 
VAR AmountScore = DIVIDE([Fraudulent_Claim_Amount], SUM(Claim_Amount), 0)
VAR RatioScore = [Potential_Loss_Ratio]
RETURN (AmountScore * 0.7) + (RatioScore * 0.3)
```
`Potential_Loss_Ratio` is defined by the identical `DIVIDE` expression as `AmountScore`, so the "70/30 weighted blend" reduces to `(X × 0.7) + (X × 0.3) = X` — it's Potential_Loss_Ratio with extra steps, not a genuine composite risk measure. **What I'd do differently:** blend in a second, actually-independent signal — fraud rate by claim volume, average fraudulent claim size relative to the specialty's norm, or something distance/history-based using the `Distance_Category` or `Patient_Claim_History` columns that already exist in the model but aren't used in the score.

**Fraud_Rank is hardcoded to one column, with a comment admitting it**
```dax
Fraud_Rank = RANKX(ALL(SynHealthClaims[Patient_State]), [Fraud_RateM], , DESC, Dense)   -- or Specialty column
```
The `-- or Specialty column` comment is a leftover note-to-self — this measure only ever ranks by Patient_State, it doesn't dynamically re-rank by whatever the report is currently sliced on. Compare to Corporate Expense Audit's `RANKX`/`ALLSELECTED` fix, which does handle that correctly. **What I'd do differently:** either parameterize the ranking dimension or build a second explicit `Fraud_Rank_Specialty` measure instead of leaving a comment as a placeholder for work that never got done.

**No explicit star schema**
The model runs on one flat fact table with Power BI's auto-generated local date tables, not an explicit Dates dimension. Fine for a 20K-row dataset with no time-intelligence needs, but a deliberate simplification worth naming rather than presenting as a finished data model.

---

## Tools Used

- **Power BI Desktop** — DAX measures, calculated columns, matrix drill-down
- **SQL Server** — staged the cleaned dataset before connecting Power BI via Power Query
- **Power Query / M** — source connection and type transformation

---

## Repo Structure

```
Healthcare_Claims_Fraud_Risk_Analysis/
├── Healthcare_Claims_Fraud_Risk_Analysis.pbix
├── screenshots/
│   ├── overview.png
│   └── risk-matrix.png
└── README.md
```

---

## Author

**Ryan Seguiro** — Data Analytics
[GitHub @nayR02](https://github.com/nayR02) · [LinkedIn](https://linkedin.com/in/ryanseguiro) · [Portfolio](ryanseguiro-portfolio.netlify.app)
