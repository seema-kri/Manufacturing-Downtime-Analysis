# Manufacturing Downtime Analysis

Estimating 392 minutes of recoverable downtime on a soft drink bottling line using Excel, Power BI and DAX.

## Table of Contents

- [Overview](#overview)
- [Business Problem](#business-problem)
- [Dataset Description](#dataset-description)
- [Tools & Technologies](#tools--technologies)
- [Project Structure](#project-structure)
- [Data Cleaning & Preparation](#data-cleaning--preparation)
- [EDA & Key Insights](#eda--key-insights)
- [Dashboard](#dashboard)
- [Key DAX Measures](#key-dax-measures)
- [Data Validation](#data-validation)
- [How to Run This Project](#how-to-run-this-project)
- [Final Recommendations / Future Work](#final-recommendations--future-work)
- [Author & Contact](#author--contact)

## Overview

The line runs at **64% efficiency** and loses **1,388 minutes** to downtime. **56%** of that downtime is operator-controllable. Cross-training operators in both directions could lift efficiency to about **71%**, saving an estimated **392 minutes**.

![Dashboard](screenshots/dashboard.png)

| KPI | Value |
|---|---|
| Line efficiency | **64%** (2,470 ideal min / 3,858 actual min) |
| Total downtime | **1,388 min** |
| Total batches | 38 |
| Avg downtime per batch | 36.5 min |
| Operator-controllable downtime | **55.9%** (776 min) |

## Business Problem

Bottling lines earn money only while they run. Unplanned stops, changeovers and shortages quietly drain output and margin. I analyzed 38 production batches from a Philadelphia soft drink line to answer three questions:

1. How efficient is the line?
2. Which downtime factors matter most?
3. Where can operators improve, and who can help whom?

## Dataset Description

| Item | Detail |
|---|---|
| Plant | Soft drink bottling line, Philadelphia (Wolf Cola case study) |
| Period | 29 Aug to 03 Sep 2024 |
| Batches | 38 |
| Products | 6 |
| Downtime factors | 12 (each flagged as operator error or not) |
| Operators | 4 (Charlie, Dee, Dennis, Mac) |
| Tables | Batches (tblProductivity), Line downtime, Downtime factors, Products |

This is sample data from a public analytics case study. It contains no real company information.

## Tools & Technologies

- **Excel:** efficiency calculation, Pareto chart, operator by factor heatmap
- **Power BI:** data model, dashboard, slicers
- **DAX:** KPI and Pareto measures
- **Git / GitHub:** version control and documentation

## Project Structure

```
Manufacturing-Downtime-Analysis/
├── README.md
├── dashboard/          Power BI file (.pbix)
├── data/               Raw data
├── Excel_Analysis/     Excel workbook
├── documentation/      Project presentation and notes
└── screenshots/        Dashboard, data model, Excel images
```

## Data Cleaning & Preparation

1. **Data model:** linked batches, downtime minutes, downtime factors and products in Power BI.
2. **Batch time:** calculated from start and end times.
3. **Downtime definition:** actual batch time minus minimum batch time.
4. **Efficiency:** minimum batch time divided by actual batch time (2,470 / 3,858 min).
5. **Reconciliation:** factor minutes matched batch downtime with 0 differences.
6. **Operator-error flag:** used to split controllable from uncontrollable causes.

![Data Model](screenshots/data_model.png)

## EDA & Key Insights

**1. Five factors cause 80% of downtime (Pareto).** Machine adjustment (332 min), machine failure (254), inventory shortage (225), batch change (160) and batch coding error (145) make up 80.4% of all lost time.

![Excel Analysis](screenshots/excel_analysis.png)

**2. Operators have opposite strengths.**

| Operator | Batch change (min) | Machine adjustment (min) | Efficiency |
|---|---|---|---|
| Charlie | 10 | 118 | 66.8% |
| Dee | 20 | 79 | 64.1% |
| Dennis | 0 | 120 | 63.2% |
| Mac | **130** | **15** | 60.9% |

Mac loses 130 of the 160 batch-change minutes but handles machine adjustment best. The other three handle batch change well and lose 79 to 120 minutes each on machine adjustment.

**3. No sustained improvement over time.** Downtime per batch peaked at 46 min on 02 Sep. The best day was 31 Aug at 24 min. 03 Sep had one batch only, so it should not be read as a trend.

## Dashboard

Power BI dashboard with slicers for Operator, Date and Product.

- **KPI cards:** line efficiency, total downtime, total batches, average downtime per batch, operator-controllable %
- **Pareto chart:** downtime minutes by factor with cumulative % line
- **Operator comparison:** efficiency by operator
- **Heatmap:** operator by factor downtime, to spot hotspots
- **Trend:** downtime per batch by day

![Dashboard](screenshots/dashboard.png)

## Key DAX Measures

```DAX
Operator-controllable % =
DIVIDE (
    CALCULATE ( SUM ( 'Line downtime'[Downtime min] ), 'Downtime factors'[Operator Error] = "Yes" ),
    SUM ( 'Line downtime'[Downtime min] )
)
```

```DAX
Cumulative % =
VAR cur = SUM ( 'Line downtime'[Downtime min] )
VAR cumul =
    CALCULATE (
        SUM ( 'Line downtime'[Downtime min] ),
        FILTER (
            ALL ( 'Downtime factors'[Description] ),
            CALCULATE ( SUM ( 'Line downtime'[Downtime min] ) ) >= cur
        )
    )
VAR total = CALCULATE ( SUM ( 'Line downtime'[Downtime min] ), ALL ( 'Downtime factors' ) )
RETURN
    DIVIDE ( cumul, total )
```

Other measures: Line Efficiency, Total Downtime, Total Batches, Avg Downtime per Batch.

## Data Validation

I recomputed every dashboard figure independently from the raw data (efficiency, totals, factor totals, operator efficiency, heatmap cells, daily trend). All values matched.

## How to Run This Project

1. Clone the repository:
   ```bash
   git clone https://github.com/seema-kri/Manufacturing-Downtime-Analysis.git
   cd Manufacturing-Downtime-Analysis
   ```
2. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
3. Open `dashboard/Dashboard.pbix`.
4. Use the slicers for Operator, Date and Product.
5. Open the workbook in `Excel_Analysis/` to see the efficiency, Pareto and heatmap work.

## Final Recommendations / Future Work

**Recommendations**

1. **Mac coaches machine adjustment.** If the other three reach Mac's 15 min, about 272 min are saved.
2. **Charlie, Dee and Dennis coach batch change.** If Mac reaches Charlie's 10 min, about 120 min are saved.
3. **Track efficiency weekly** after training and rerun the Pareto to confirm the gain.

**Estimated impact:** 2,470 / (3,858 minus 392) = **71.3% efficiency**, up from 64%.

> This is an estimate. It assumes weaker operators reach the level of their strongest peer on each factor (machine adjustment 15 min, batch change 10 min). It ignores product mix (the 2 L product has a longer minimum batch time). Mac and Dennis each ran 8 batches, so operator gaps are a starting point for coaching, not a performance judgment.

**Future work**

1. Control for **product mix** when comparing operators.
2. Collect more batches per operator before drawing operator-level conclusions.
3. Add **shift and time of day** analysis to see when downtime clusters.
4. Add **cost impact** (lost output value per downtime minute) to rank fixes by money.
5. Build a **weekly efficiency tracker** to measure the real gain after cross-training.
6. Test **predictive maintenance** signals for machine failure, the second largest factor.

## Author & Contact

**Seema Kumari**
Data Analyst | Business Intelligence & Microsoft Fabric

- 💼 LinkedIn: [linkedin.com/in/seema-kumari-375763308](https://linkedin.com/in/seema-kumari-375763308)
- 📧 Email: [seemakri136@gmail.com](mailto:seemakri136@gmail.com)
- 🐙 GitHub: [github.com/seema-kri](https://github.com/seema-kri)
- 💻 LeetCode: [leetcode.com/u/seemakri136/](https://leetcode.com/u/seemakri136/)

---

⭐ If you found this project useful, consider giving it a star. It helps others discover it.
