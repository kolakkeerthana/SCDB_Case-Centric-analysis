# Supreme Court Case Operations & Decision Analytics — End-to-End BI Solution
To understand the judgement patterns overtime


Problem Statement
How has the Supreme Court's caseload, decision-making, case-processing time, and voting patterns changed over time, and where are there notable trends or anomalies that warrant further investigation?

Source : http://scdb.wustl.edu/data.php


the process would follow this way 
SCDB Data
       ↓
Data Profiling
       ↓
Data Cleaning & Validation
       ↓
SQL Analytical Layer
       ↓
Statistical / Exploratory Analysis
       ↓
Tableau Dashboards
       ↓
Insights & Recommendations

Now to proceed with the problem statement we would answering the folowing questions
Question 1 — Caseload

How has the Court's caseload changed over time?

Question 2 — Case processing

How long does it take for cases to move from argument to decision?

Question 3 — Case outcomes

What types of cases and issue areas have different decision outcomes?

Question 4 — Decision alignment

How frequently are decisions unanimous versus divided, and how does this vary over time and issue area?

Question 5 — Anomalies and data quality

Where do we see unusual processing times, voting patterns, or data-quality problems?

Data Profiling
<img width="2266" height="702" alt="image" src="https://github.com/user-attachments/assets/a1d47a29-1ecd-4daf-815b-afe99c73066a" />

Data cleaning



Performing EDA

How has Supreme Court case volume changed over time?

# ⚖️ Supreme Court Case Operations & Decision Analytics — End-to-End BI Solution

📌 Introduction

This project explores the Supreme Court Database (SCDB) Case-Centered dataset, sourced from [scdb.wustl.edu](http://scdb.wustl.edu/data.php), covering Supreme Court decisions from 1946 onward. The dataset was ingested in Python (pandas), profiled, cleaned, and enriched with derived features, then prepared for exploratory analysis and Tableau dashboarding.

Our objective is to understand how the Court's caseload, decision-making, case-processing time, and voting patterns have changed over time, and to surface trends or anomalies worth further investigation.

**Guiding questions:**
1. **Caseload** — How has the Court's caseload changed over time?
2. **Case processing** — How long does it take for cases to move from argument to decision?
3. **Case outcomes** — What types of cases and issue areas have different decision outcomes?
4. **Decision alignment** — How frequently are decisions unanimous versus divided, and how does this vary over time and issue area?
5. **Anomalies and data quality** — Where do we see unusual processing times, voting patterns, or data-quality problems?

**Pipeline:**
```
SCDB Data
   ↓
Data Profiling
   ↓
Data Cleaning & Validation
   ↓
SQL Analytical Layer
   ↓
Statistical / Exploratory Analysis
   ↓
Tableau Dashboards
   ↓
Insights & Recommendations
```

🧹 Data Cleaning & Preparation

1. **Column Review & Selection**
   Inspected all 53+ columns in the raw case-centered file for relevance to the analysis. Narrowed the working table to a focused `case_analysis` set — case identifiers, dates, decision/vote codes, and issue classification — dropping citation and administrative fields not needed for the caseload/processing/voting questions.

2. **Uniqueness Checks**
   Verified that `caseId`, `docketId`, `caseIssuesId`, and `voteId` behave as expected identifiers, confirming the "1 row = 1 case" grain before aggregating, so that later relationships (docket → issues → votes) aren't double-counted.

3. **Date Type Conversion & Validation**
   Converted `dateDecision`, `dateArgument`, and `dateRearg` from text to proper datetime types. Checked the min/max range of each, and flagged invalid records where the argument date fell after the decision date.

4. **Data Profiling & Missing Values**
   Built a profiling table (data type, non-null count, missing %, unique values) for both the raw file and the working table, sorted by missing percentage to prioritize which fields needed attention.

5. **Feature Engineering**
   - `vote_margin` = `majVotes` − `minVotes`
   - `majority_vote_pct` = `majVotes` / (`majVotes` + `minVotes`) × 100
   - `decision_year` = year extracted from `dateDecision`
   - `processing_days` = `dateDecision` − `dateArgument`, with a negative-day check to confirm chronological validity
   - `processing_category` = binned version of `processing_days` (0–30, 31–90, 91–180, 181–365, 365+ days)

6. **Decoding Categorical Codes**
   The SCDB stores many fields as numeric codes. Each was cross-checked against the official SCDB codebook and mapped to a human-readable `_name` column, including:
   - `decisionType` → e.g. Opinion of the Court, Per Curiam, Equally Divided Vote
   - `issueArea` → e.g. Criminal Procedure, Civil Rights, First Amendment, Economic Activity
   - `jurisdiction` → e.g. Certiorari, Appeal, Original Jurisdiction
   - `caseDisposition` → e.g. Affirmed, Reversed, Vacated and Remanded
   - `partyWinning` → Petitioner Won / Petitioner Did Not Win / Outcome Unclear
   - `decisionDirection` → Conservative / Liberal / Unspecifiable
   - `decisionDirectionDissent`, `declarationUncon`, `precedentAlteration`, `voteUnclear`

   Each mapping was verified by comparing the original code against its decoded label before adopting it into the analytical table.

7. **Final Analytical Table**
   The cleaned, enriched, and decoded table (`SupremeCourtCaseAnalysis`) was exported for exploratory analysis in Python and for building visualizations in Tableau.

Data is now clean and ready for exploratory analysis.

🔍 Exploratory Data Analysis (EDA)

**1. Caseload Over Time**
Grouped cases by `decision_year` and counted unique `caseId`s to trace how the Court's annual case volume has shifted across decades.
🎯 *Insight: Caseload is not flat over time — certain periods show markedly higher or lower volume.*

**2. Case Processing Time**
Calculated `processing_days` between argument and decision, summarized overall and by `decision_year` (count, mean, median, min, max) and by `issueArea_name` (count, mean, median).
🎯 *Insight: Typical processing time varies meaningfully by issue area, and outlier cases can take far longer than the median.*

**3. Voting Pattern Analysis**
Summarized `majVotes`, `minVotes`, `vote_margin`, and `majority_vote_pct` overall and by issue area to compare how consensus-driven vs. divided decisions are across topics.
🎯 *Insight: Some issue areas trend toward narrower vote margins (more divided courts) than others.*

**4. Case Outcomes**
Tabulated case counts by `caseDisposition_name` (Affirmed, Reversed, Vacated and Remanded, etc.) and by `partyWinning_name` (Petitioner Won / Did Not Win / Outcome Unclear).
🎯 *Insight: Petitioner win rate varies widely by issue area — from over 70% in Private Action and Attorneys cases down to under 50% in Interstate Relations and Miscellaneous cases.*

**5. Petitioner Win Rate by Issue Area**
Computed `total_cases`, `petitioner_wins`, and `petitioner_win_rate` grouped by `issueArea_name`, sorted to rank issue areas from most to least favorable for petitioners.

📊 Dashboard (Tableau)

**KPIs:**
- Total Cases
- Average Processing Time (days)
- Average Vote Margin
- Petitioner Win Rate

**Visuals:**
- Case Volume by Year (trend line)
- Processing Time by Year and Issue Area
- Vote Margin / Majority % by Issue Area
- Case Disposition & Petitioner Outcome Breakdown
- Petitioner Win Rate by Issue Area

**Business / Research Insights:**
- Track how caseload shifts over time to contextualize staffing and scheduling of the Court's term.
- Flag issue areas and years with unusually long processing times for further investigation.
- Monitor vote margin trends to understand where the Court is more divided versus unanimous, and how that shifts by issue area over time.
- Use petitioner win-rate patterns by issue area to inform case-outcome research and identify anomalies worth a closer look.
- Continue validating SCDB code mappings against the official codebook whenever the dataset is refreshed, to keep decoded labels accurate.

## About
An end-to-end BI project cleaning, analyzing, and visualizing Supreme Court Database (SCDB) case data in Python to uncover caseload, processing-time, and voting-pattern insights, feeding into Tableau dashboards.

**Data Source:** [Supreme Court Database (SCDB)](http://scdb.wustl.edu/data.php)
<img width="2110" height="1112" alt="image" src="https://github.com/user-attachments/assets/748bc0b6-12db-4149-8981-b6fb60a56af8" />



