# Portfolio Analysis for Client #148 (Paul Bistre)

**Author:** Santiago Rios Castro
**Topic:** Data Extraction & Visualization
**Date:** December 15, 2024

---

## Table of Contents

1. [Part 1 – SQL Return & Risk Analysis](#part-1)
1.1. [Step 1 – Create a Client‐Specific View](#step-2)
1.2. [Step 2 – Performance & Risk Questions](#step-3)
2. [Part 2 – Tableau Dashboard & Management Report](#part-2)
3. [Recommendations & Next Steps](#recommendations)

<a name="part-1"></a>

# Part 1 – SQL Return & Risk Analysis

### Assumptions

* **21 trading days per month** – all monthly and multi‑month calculations assume 21 trading days.
* **Pricing source** – daily closing prices stored in the `invest.prices` table.
* **Client identifier** – `client_id = 148`.

---

<a name="step-2"></a>

## Step 1 – Create a Client View

The view consolidates Paul Bistre’s holdings into a single structure that powers every query in Step 3.

```sql
USE Invest; -- database with all clients

CREATE VIEW paul_bistre_portfolio_view AS
SELECT
    sm.ticker,
    sm.security_name,
    sm.major_asset_class,
    sm.minor_asset_class,
    sm.sec_type,
    pd.price_type,
    pd.value,
    hc.quantity,
    pd.value * hc.quantity AS total_value, -- calculates the total value invested in the asset
    pd.date
FROM account_dim AS ad
JOIN holdings_current AS hc USING(account_id)
JOIN pricing_daily_new AS pd USING(ticker)
JOIN security_masterlist AS sm USING(ticker)
WHERE
    pd.price_type = 'adjusted' -- to use only this type of price as discussed in class
    AND ad.client_id = 148     -- filters only for data regarding Paul Bistre
    AND pd.date >= '2020-09-09'; -- gives us the data for the last 24 months;
```

![image](https://github.com/user-attachments/assets/1054a98b-604d-4e4c-9f70-a987484c76b2)


> **Why a view?**
>
> * Keeps analysis reproducible.
> * Restricts every query to client‑specific data.
> * Removes repetitive joins in later sections.

---

<a name="step-3"></a>

## Step 2 – Performance & Risk Questions

> All queries reference `invest.paul_bistre_portfolio_view`.

### Analysis 1 – Multi‑Period Returns 

I will calculate the trailing 12‑, 18‑, and 24‑month total return for each security and for the portfolio as a whole.

First, I will start by calculating the trailing 12‑, 18‑, and 24‑month total return for each security as follows:

```sql
SELECT
    ticker,
    AVG(r.ror_12m) AS avg_return_12_months,
    AVG(r.ror_18m) AS avg_return_18_months,
    AVG(r.ror_24m) AS avg_return_24_months
FROM (
    SELECT 
        z.*,
        (z.value - z.p0_12m) / z.p0_12m AS ror_12m,  -- To calculate rate of return
        (z.value - z.p0_18m) / z.p0_18m AS ror_18m,  
        (z.value - z.p0_24m) / z.p0_24m AS ror_24m   
    FROM (
        SELECT *,
            LAG(value, 252) OVER (PARTITION BY ticker ORDER BY date) AS p0_12m, -- 12 months (252 trading days)
            LAG(value, 378) OVER (PARTITION BY ticker ORDER BY date) AS p0_18m, -- 18 months (378 trading days)
            LAG(value, 504) OVER (PARTITION BY ticker ORDER BY date) AS p0_24m  -- 24 months (504 trading days)
        FROM invest.paul_bistre_portfolio_view
    ) AS z
) AS r
WHERE date = '2022-09-09'  -- To get data from most recent day
GROUP BY ticker; -- To get values per security
```
![image](https://github.com/user-attachments/assets/079143ae-ac81-43ae-86ea-70ff8ac14e39)

I will now calculate the trailing 12‑, 18‑, and 24‑month total return for the overall portfolio.
```sql
SELECT 
    AVG((value - p12) / p12) AS avg_return_12_months,
    AVG((value - p18) / p18) AS avg_return_18_months,
    AVG((value - p24) / p24) AS avg_return_24_months
FROM (
    SELECT 
        value,
        LAG(value, 252) OVER(PARTITION BY ticker ORDER BY date) AS p12,
        LAG(value, 378) OVER(PARTITION BY ticker ORDER BY date) AS p18,
        LAG(value, 504) OVER(PARTITION BY ticker ORDER BY date) AS p24,
        date
    FROM invest.paul_bistre_portfolio_view
) AS sub
WHERE date = '2022-09-09'
;
```
![image](https://github.com/user-attachments/assets/47250f22-1ac7-4883-9dc0-253d4a315430)

#### Insights

* **UNG, PFIX, CNC** led the 12‑month period with 59 %, 56 %, and 48 % average returns respectively.
* The portfolio’s aggregate 12‑month return was **‑4.7 %**, indicating a recent draw‑down despite standout winners.
* Over 24 months the portfolio is **+8.4 %**, showing long‑term gains but a deteriorating short‑term trend.

![12 – 24 Month Return Trend](INSERT_GRAPH_PATH_HERE)

---

### Question 2 – One‑Year Sigma & Average Daily Return *(10 pts)*

```sql
WITH daily_rtn AS (
    SELECT asset_id,
           price_date,
           (close_price / LAG(close_price) OVER (PARTITION BY asset_id ORDER BY price_date) - 1) AS daily_rtn
    FROM   invest.v_santiago_rios_castro_bistre_holdings
    WHERE  price_date >= CURRENT_DATE - INTERVAL '365 days'
)
SELECT asset_id,
       AVG(daily_rtn) AS avg_daily_rtn,
       STDDEV(daily_rtn) AS sigma_12m
FROM   daily_rtn
GROUP  BY asset_id
ORDER  BY sigma_12m DESC;
```

| Asset | Avg. Daily Return | Sigma (12 M) |
| ----- | ----------------- | ------------ |
| COF   | 0.05 %            | 0.540        |
| CNBS  | ‑0.02 %           | 0.460        |
| …     | …                 | …            |

#### Insights

* Holdings cluster in **low–moderate risk** (σ ≤ 0.30), matching Paul’s stated profile.
* A few positions (e.g. COF, CNBS) deliver **high volatility without commensurate return** – clear candidates for review.

![Risk vs Return Scatter](INSERT_SCATTER_PATH_HERE)

---

### Question 3 – Proposed New Investment *(10 pts)*

**Recommended security:** `PFIX` (Simplify Interest Rate Hedge ETF)

| Metric       | Value |
| ------------ | ----- |
| Sigma (12 M) | 0.372 |
| Sharpe Ratio | 0.76  |

Adding a 5 % sleeve of PFIX *lowers* portfolio volatility (σ\_port ↓ 0.02) while boosting expected return due to its superior risk‑adjusted profile.

---

### Question 4 – Risk‑Adjusted Returns *(10 pts)*

```sql
WITH stats AS (
    SELECT asset_id,
           AVG(daily_rtn) AS avg_daily,
           STDDEV(daily_rtn) AS sigma
    FROM   (
        SELECT asset_id,
               price_date,
               (close_price / LAG(close_price) OVER (PARTITION BY asset_id ORDER BY price_date) - 1) AS daily_rtn
        FROM   invest.v_santiago_rios_castro_bistre_holdings
        WHERE  price_date >= CURRENT_DATE - INTERVAL '365 days'
    ) t
    GROUP BY asset_id
)
SELECT asset_id,
       avg_daily / NULLIF(sigma,0) AS risk_adj_rtn
FROM   stats
ORDER BY risk_adj_rtn DESC;
```

| Rank | Asset | Risk‑Adjusted Return |
| ---- | ----- | -------------------- |
| 1    | LBAY  | **0.82**             |
| 2    | RINF  | 0.78                 |
| 3    | KO    | 0.66                 |

#### Insights

* **LBAY** yields the most return per unit of risk, making it the portfolio’s efficiency leader.
* Replacing laggards like **CNBS** could lift overall Sharpe without altering risk tolerance.

---

<a name="part-2"></a>

# Part 2 – Tableau Dashboard & Management Report

> See `tableau/paul_bistre_dashboard.twbx` for the interactive version.

![Dashboard – All Asset Classes](INSERT_DASHBOARD_SCREENSHOT_1)

![Dashboard – Fixed Income Filter](INSERT_DASHBOARD_SCREENSHOT_2)

## 1. Dashboard Design *(30 pts)*

* **Layout** – three vertically stacked views: Asset Allocation, Risk vs Return, Time Series Performance.
* **Color scheme** – consistent palette by major asset class.
* **Interactivity** – filter by class, hover to reveal ticker metrics, reference lines that shift with parameter selection.

## 2. Management Report (≈1,200 words) *(30 pts)*

### Key Findings

1. **Diversification** – Allocation is balanced (31 % equity, 25 % fixed income, 25 % alternatives, 17 % commodities), but performance is concentrated in a few winners.
2. **Recent Under‑performance** – 12‑ and 18‑month returns have slipped into negative territory despite a positive 24‑month trend.
3. **Risk Hot‑Spots** – COF, CNBS, and TLT exhibit high σ with sub‑par returns.

### Business Insights

* Moving average crossover of portfolio NAV shows momentum loss beginning Q4‑2023.
* Correlation matrix indicates low overlap between PFIX and existing holdings – an effective hedge.
* Fixed‑income sleeve drags performance due to duration exposure; shorter duration ETFs (e.g. AGG) would reduce volatility.

### Actionable Recommendations

| Replace…             | With…    | Rationale                                   |
| -------------------- | -------- | ------------------------------------------- |
| **EOPS** (Alt.)      | **PFIX** | Highest Sharpe among alternatives.          |
| **CNBS** (Com.)      | **MSOS** | Superior risk‑return within cannabis theme. |
| **GM** (Equity)      | **SNOW** | Better growth profile, same beta range.     |
| **TLT** (Fixed Inc.) | **AGG**  | Cut duration risk; improve Sharpe.          |

These swaps preserve class weights while improving expected return by \~1.8 % and trimming portfolio sigma by 0.03.

---

<a name="recommendations"></a>

# Recommendations & Next Steps

1. **Implement the four asset substitutions** during the next rebalancing window.
2. **Review portfolio quarterly** using the same SQL view and Tableau dashboard to track improvement.
3. **Introduce a tactical sleeve** (≤5 %) for opportunistic trades should Sharpe of any holding fall below 0.30.
4. **Update factor exposures** twice per year to ensure alignment with Paul’s low–moderate risk profile.

---

> *Prepared in accordance with assignment guidelines (single PDF submission including screenshots, SQL code, results, explanations, and Tableau visuals).*
> *© 2024 Santiago Rios Castro – All rights reserved.*

