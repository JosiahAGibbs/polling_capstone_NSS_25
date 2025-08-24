# Forecast Face-Off: Markets vs. Polls
Capstone project for Nashville Software School (Aug ’25).

**Table of Contents**
- [Power BI Dashboard](#power-bi-dashboard)
- [Motivation](#motivation)
- [Questions](#questions)
- [Statisitical Concepts](#statisitical-concepts)  
- [Sourcing and Cleaning Data](#sourcing-and-cleaning-data)
- [Problems](#problems)
- [Tech Stack](#tech-stack)
- [Conclusion](#conclusion)


## Power BI Dashboard

## Motivation
I have always been fascinated by the horse race of political elections. In my lifetime, I’ve seen two of the most polarizing outcomes—the 2016 and 2024 elections. Although the gap between polls and outcomes narrowed between 2016 and 2024, confidence in polling was shaken in 2016 and again in 2020. Prediction markets have garnered interest over the past decade as a complementary signal for gauging voter sentiment, and there is ongoing research and commentary on their value. Markets move with **money**; polls move with **respondents**. In prediction markets, there is real money at stake, while polls rely on evolving data-collection methods (online panels, phone interviews).

This project tests whether **market prices** offer earlier or better signals than **polling averages** in the 2024 U.S. presidential race, focusing on 7 swing states (AZ, GA, MI, NC, NV, PA, WI). Beyond accuracy, the focus is on **when** each signal moves and how often they **agree**—insights that matter for analysts, journalists, prediction-market traders, and decision-makers who must act before official results.

## Questions
- Accuracy: Who called state winners better in the final week—markets or polls?

- Calibration: Whose probabilities were better calibrated (Brier score)?

- Timing: Do markets lead or lag poll movements, and by how many days?

- Agreement: How often do markets and polls pick the same winner?

- Tilt & Separation: In which states do markets lean more GOP/Dem relative to polls, and by how much (pp)?

## Sourcing and Cleaning Data

The market data was sourced from the Polymarket API; polling tables were scraped from RealClearPolitics. Both were gathered with requests and BeautifulSoup in Jupyter, cleaned with pandas, and exported to CSVs used by the Power BI. Feature engineering (MAE, signed error, lead–lag, final-week snapshot) was performed in Python and Power Query/DAX. Brier and outcome analysis were completed in Power BI.

## Statisitical Concepts 

- MAE (Mean Absolute Error)
    - Deteremines how far off on average a data topic is
    - Gives us magnitude without assuming a direction
    - In this project, MAE measures how far apart markets and polls were on a given day

- Signed Error
    - Assumes direction of the difference: Market vs. Poll
        -Positive = market higher than poll; negative = market lower
    - A way of determining tilt/"bias"

- Agreement %
    - Establishes share of days where markets and polls picked the same winner
    - Are markets and polls telling the same story?
    - 62.4% was the discovered agreement %
        - This means 3/5 of the time markets and polls agreed, and the remaining 2/5 they divereged

- Brier Score
    - Tests the quality of a probability by scoring it (Predicted probability - actual outcome)^2
    - Ranges 0 to 1, the lower the better
        -0.00 = perfect, 0.25 ≈ coin flip, 1.00 = confidently wrong
    - Rewards being "right" and calibrated, punshes overconfidence.

- Lead-Lag & Shift Days
    - Do markets move before polls (or after), and by how many days?
        - Negative shift = markets lead (move earlier)
        - Positive shift = markets lag (move later)

## Problems

- Tons of data to clean and sort through: This project was less “make a pretty chart” and more “tame a wild data monster.” Between Polymarket’s API outputs and RCP’s HTML tables, I was drowning in messy, inconsistent, sometimes-missing data.

- Working with APIs: Rate limits/timeouts happen. I added simple retries and cached responses so re-runs didn’t hammer endpoints.

- Scrape quirks: RCP tables shift columns and formats; I had to harden the parser and add guards for “47%”→47.0, date ranges, and edge-case headers

- Mixed scales (0–1 vs 0–100): I accidentally mixed units and MAE ballooned. Fixed by standardizing to percentage points (0–100) everywhere.

- Type errors: “AVERAGE can’t work with values of type String.” Yep—some columns were text. I coerced with VALUE() in PQ/DAX and stopped leaning on implicit aggregation.

- Ambiguous relationships: Multiple active paths = blanks/dupes. I trimmed it down to one active, one-way link: States[StateKey] → fact[StateKey].

- Parameter wiring: My threshold slicer was targeting the column instead of the Value measure. Added a safe default and referenced the slider’s Value in the masking logic.

- Flatline measures: Same number for every state = no state context. I enforced it with TREATAS/LOOKUPVALUE and a hardened StateKey.
  
- Alignment for analysis: Lead–lag requires consistent windows; outcomes need a clear “final 7 days.” I computed those snapshots per state off each state’s last valid date.

## Tech Stack

- Power BI Desktop (DAX, Power Query/M) for modeling & visualization

- Python (pandas/requests/beautiful soup/Jupyter) gathering data, quick tables, csv generataion, MAE, and lead/lag preprocessing

- GitHub for generating this write-up

# Conclusion

## What I found

- At the finish line, polls won. Final-week winner accuracy: 86% polls vs 71% markets. Brier: 0.16 (polls) vs 0.17 (markets).

- Markets often moved earlier, but the signal was modest. Examples: AZ (Rep −6d, r≈0.57), NV (Rep −7d, r≈0.34), PA (Rep +7d lag, r≈0.48), MI (both −9d lead, r≈0.3). Helpful directionally, not a guarantee of better final calls.

- Markets leaned GOP relative to polls in most battlegrounds. You can see it in the Page-1 KPIs: Rep MAE 11.77 pp vs Dem MAE 7.94 pp, Dem signed −1.87 pp, Rep signed +9.20 pp. Markets and polls agreed only 62.40% of days.

## How to use it

- Use markets as an early directional read.

- Use polls as the better-calibrated finish-line source (in this sample).

- Expect state variation: markets did better in GA/NC; polls did better in MI/WI/NV; AZ was basically a tie.

## Next steps

- Go beyond these seven battlegrounds and this one cycle.

- Weight polls by quality/recency.

- Difference/detrend before lead–lag; try hierarchical/pooled models.

- Track out-of-sample Brier over rolling windows.
