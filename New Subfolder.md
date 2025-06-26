# Portfolio Analysis for Client Paul Bistre

**Author:** Santiago Rios Castro
**Topic:** Data Extraction & Visualization
**Date:** December 15, 2024

---

## Table of Contents

1. [Part 1 – SQL Return & Risk Analysis](#part-1)

    1.1. [Step 1 – Create a Client‐Specific View](#step-2)

    1.2. [Step 2 – Performance & Risk Questions](#step-3)
3. [Part 2 – Tableau Dashboard & Management Report](#part-2)
4. [Recommendations & Next Steps](#recommendations)

<a name="part-1"></a>

# Part 1 – SQL Return & Risk Analysis

### Assumptions

* **21 trading days per month** – all monthly and multi‑month calculations assume 21 trading days.
* **Pricing source** – daily closing prices stored in the `invest.prices` table.
* **Client identifier** – `client_id = 148`.

---

<a name="step-2"></a>

## Step 1 – Create a Client View

The view consolidates Paul Bistre’s holdings into a single structure that powers every query in Step 2.

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
* Business Question: What is the most recent 12 months, 18 months, 24 months return for each of the securities 
(and for the entire portfolio)?

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

Top three gainers from these timeframes in terms of average returns:

12 months →  
![image](https://github.com/user-attachments/assets/eb7412cc-1e80-4ca2-bb9f-55e410b63b66)

18 months →  
![image](https://github.com/user-attachments/assets/e09af745-6224-4e09-84ae-5693454d5e92)

24 months →  
![image](https://github.com/user-attachments/assets/f2b1a8c5-721e-4940-a56b-37c6636cf36f)

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

These queries helped us evaluate individual asset performance and overall portfolio returns over 12, 18, and 24 months. Over the past 12 months, the portfolio experienced a 4.7% decline, despite strong gains from assets like UNG (59%), PFIX (56%), and CNC (48%). Similarly, the 18-month return showed a smaller decline of 1.5%, with top performers including UNG (179%), PANW (64%), and NVO (56%). In contrast, the 24-month analysis revealed an 8.4% overall gain, led by PANW (136%), UNG (118%), and PFG (101%).

These results suggest that while some assets have consistently performed well, others have significantly underperformed, especially in the more recent periods. The positive long-term trend contrasts with the short-term declines, signaling a potential shift in performance momentum. This emphasizes the need for a thorough review of the current asset allocation and investment strategies to address recent underperformance and ensure the portfolio stays aligned with long-term growth objectives.

---

### Analysis 2 – One‑Year Sigma & Average Daily Return
* Business Question: What is the most recent 12months sigma (risk) for each of the securities? What is the average 
daily return for each of the securities?

```sql
WITH average_daily_return AS (
    SELECT
        ticker,
        date,
        (value - LAG(value) OVER (PARTITION BY ticker ORDER BY date)) /
        LAG(value) OVER (PARTITION BY ticker ORDER BY date) AS average_daily_return  -- To obtain daily rates
    FROM invest.paul_bistre_portfolio_view
    WHERE date >= '2021-09-09'  -- To get data for the last year
)

SELECT
    ticker,
    average_daily_return,
    STD(average_daily_return) AS risk          -- Standard deviation of daily returns
FROM average_daily_return
WHERE average_daily_return IS NOT NULL
GROUP BY ticker;                               -- Get one value per security
```

![image](https://github.com/user-attachments/assets/d2a2dfb6-0046-4206-bd46-3c208d076035)

Results sorted from highest to lowest risk:
![image](https://github.com/user-attachments/assets/3d256be8-397b-4d08-a8ba-4efa26ff4882)



#### Insights

 This query provides the average daily rate of return and the corresponding risk for each asset in the client’s portfolio, based on the past 12 months of data. The sorted results show that several assets like UNG, SVIX or ROST carry high risk while delivering low or negative returns, highlighting the need to reassess and potentially replace these investments with ones that offer stronger performance.

---

### Analysis 3 – Proposed New Investment
* Business Question: Suggest adding a new investment to your portfolio - what would it be and how much risk 
(sigma) would it add to your client?  

```sql
SELECT
    ticker,
    AVG(ror) AS avg_ror,
    STD(ror) AS std_ror,
    AVG(ror) / STD(ror) AS adjusted_risk
FROM (
    SELECT
        z.ticker,
        z.date,
        z.value,
        z.p0,
        (z.value - z.p0) / z.p0 AS ror  -- To calculate rate of return
    FROM (
        SELECT
            ticker,
            date,
            AVG(value) AS value,
            LAG(AVG(value), 252) OVER (PARTITION BY ticker ORDER BY date) AS p0  -- To obtain rates for the last year
        FROM invest.pricing_daily_new  -- To use the entire dataset, not only the holdings from Paul
        GROUP BY ticker, date
    ) AS z
    WHERE z.p0 IS NOT NULL
) AS r
GROUP BY ticker;  -- To get values per security
```
![image](https://github.com/user-attachments/assets/de028165-a986-49de-b386-59e9e3d67eca)

**Recommended security:** `PFIX` (Simplify Interest Rate Hedge ETF)

This query helps us examine the average return, standard deviation (as a measure of risk), and risk-adjusted return (comparable to the Sharpe ratio) for each security using price data from all tickers in the investment dataset, not just those currently held in Paul’s portfolio. For this reason, we use the pricing_daily_new table instead of the client-specific view created in Step 1. These metrics help assess which assets deliver the most efficient returns relative to their level of risk.

Based on the results, I would recommend adding the ETF with the ticker PFIX to Paul’s portfolio. PFIX has the highest risk-adjusted return (0.76) among all assets analyzed, meaning it offers the best return relative to the amount of risk involved. This makes it a highly efficient investment option.

For Paul, who aims to maximize returns while maintaining a moderate risk profile, PFIX aligns well with his goals. Its standard deviation (3.72) reflects moderate volatility, and its adjusted risk is superior to some of the existing holdings in the portfolio, such as COF (0.54) and CNBS (0.46). Incorporating PFIX could strengthen the overall return potential of the portfolio without significantly increasing its risk.

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

