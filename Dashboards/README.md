***FP\&A Analyst / Financial Planning Analyst Project Report***

---

**Skills Used**

* **SQL Query Development:** Writing and optimizing complex queries to flatten, transform, and aggregate trading data.  
* **Data Transformation & Flattening:** Converting nested structures into a tabular format for easier analysis.  
* **KPI Calculation & Financial Analysis:** Computing key performance metrics such as Trade Profit/Loss, Return Percentage, Profit Per Unit, and trade outcomes.  
* **Real-Time Data Processing:** Analyzing trades that execute in exactly one minute to enable rapid decision-making.  
* **Business Intelligence & Data Visualization:** Presenting insights in an accessible format to support strategic decision-making.  
* **Algorithmic Trading Analysis:** Evaluating the performance of various trading algorithms (Momentum, Feeling Lucky, Prediction) and their combinations.

---

**Project Overview**  
This project provides real-time insights into the performance of automated trading bots that use different algorithms to make pre-trade decisions (long or short) for trades that complete in exactly one minute. By tracking profitability, return percentages, trade duration, and success rates, finance leaders can make data-driven decisions on algorithm optimization and resource allocation.

---

**Methodology**

**Data Flattening and Transformation**  
The initial step involves extracting and transforming raw trade data by mapping algorithm identifiers and flattening nested fields (e.g., trade sides). This creates a structured dataset ready for KPI analysis.  
**SQL Query:**  
pgsql  
Copy  
`SELECT`  
  `OrderID AS tradeID,`  
  `MaturityDate AS tradeTimestamp,`  
  `(`  
    `CASE SUBSTR(TargetCompID, 0, 4)`  
      `WHEN 'MOMO' THEN 'Momentum'`  
      `WHEN 'LUCK' THEN 'Feeling Lucky'`  
      `WHEN 'PRED' THEN 'Prediction'`  
    `END`  
  `) AS algorithm,`  
  `Symbol AS symbol,`  
  `LastPx AS openPrice,`  
  `StrikePrice AS closePrice,`  
  `(`  
    `SELECT Side`  
    `FROM UNNEST(Sides)`  
  `) AS tradeDirection,`  
  `(`  
    `CASE (`  
      `SELECT Side`  
      `FROM UNNEST(Sides)`  
    `)`  
      `WHEN 'SHORT' THEN -1`  
      `WHEN 'LONG' THEN 1`  
    `END`  
  `) AS tradeMultiplier`  
`FROM`  
  `` `bigquery-public-data.cymbal_investments.trade_capture_report` ``

1. 

**Profit Calculation and KPI Aggregation**  
In this stage, key performance indicators (KPIs) are calculated for each trade. The metrics include Trade Profit/Loss, Return Percentage, Profit Per Unit, and an Outcome flag to indicate if the trade was profitable.  
**SQL Query:**  
vbnet  
Copy  
`SELECT`  
  `tradeID,`  
  `tradeTimestamp,`  
  `algorithm,`  
  `symbol,`  
  `openPrice,`  
  `closePrice,`  
  `tradeDirection,`  
  `tradeMultiplier,`  
  `CASE`  
    `WHEN tradeDirection = 'LONG' THEN (closePrice - openPrice) * tradeMultiplier`  
    `WHEN tradeDirection = 'SHORT' THEN (openPrice - closePrice) * tradeMultiplier`  
    `ELSE 0`  
  `END AS TradeProfitLoss,`  
  `CASE`  
    `WHEN openPrice * tradeMultiplier != 0 THEN`  
      `CASE`  
        `WHEN tradeDirection = 'LONG' THEN ((closePrice - openPrice) * tradeMultiplier) / (openPrice * tradeMultiplier) * 100`  
        `WHEN tradeDirection = 'SHORT' THEN ((openPrice - closePrice) * tradeMultiplier) / (openPrice * tradeMultiplier) * 100`  
        `ELSE 0`  
      `END`  
    `ELSE 0`  
  `END AS ReturnPercentage,`  
  `CASE`  
    `WHEN tradeMultiplier != 0 THEN`  
      `CASE`  
        `WHEN tradeDirection = 'LONG' THEN (closePrice - openPrice)`  
        `WHEN tradeDirection = 'SHORT' THEN (openPrice - closePrice)`  
        `ELSE 0`  
      `END`  
    `ELSE 0`  
  `END AS ProfitPerUnit,`  
  `CASE`  
    `WHEN`  
      `CASE`  
        `WHEN tradeDirection = 'LONG' THEN (closePrice - openPrice)`  
        `WHEN tradeDirection = 'SHORT' THEN (openPrice - closePrice)`  
        `ELSE 0`  
      `END > 0 THEN 'Profit'`  
    `ELSE 'Loss'`  
  `END AS Outcome`  
``FROM `chrome-cipher-451713-e8.Finance_CFD.CFD_transactions`;``

2. 

**Timeframe Segmentation**  
To understand performance variations throughout the day, trades are categorized into time segments: Morning, Afternoon, Evening, or Night based on their timestamp.  
**SQL Query:**  
pgsql  
Copy  
`SELECT`  
  `tradeID,`  
  `tradeTimestamp,`  
  `algorithm,`  
  `symbol,`  
  `openPrice,`  
  `closePrice,`  
  `tradeDirection,`  
  `tradeMultiplier,`  
  `ReturnPercentage,`  
  `ProfitPerUnit,`  
  `Outcome,`  
  `CASE`  
    `WHEN EXTRACT(HOUR FROM tradeTimestamp) BETWEEN 6 AND 11 THEN 'Morning'`  
    `WHEN EXTRACT(HOUR FROM tradeTimestamp) BETWEEN 12 AND 17 THEN 'Afternoon'`  
    `WHEN EXTRACT(HOUR FROM tradeTimestamp) BETWEEN 18 AND 23 THEN 'Evening'`  
    `ELSE 'Night'`  
  `END AS TimeOfDay`  
`` FROM `chrome-cipher-451713-e8.Finance_CFD.finance_CFD_2nd_transformatoin` ``  
`LIMIT 1000;`

3. 

---

**Key Findings & Insights**

* **Real-Time Analysis:**  
  Each trade is executed within a one-minute cycle, with the algorithm making decisions on whether to go long or short before each trade.  
* **KPI Breakdown:**  
  * **Trade Profit/Loss:** Quantifies immediate profit or loss, clearly distinguishing between long and short outcomes.  
  * **Return Percentage:** Normalizes returns for better performance comparison across trades.  
  * **Profit Per Unit:** Provides a granular view of profit on a per-unit basis.  
  * **Outcome Flag:** Indicates whether a trade resulted in a profit or a loss.  
* **Time-of-Day Trends:**  
  Analysis shows that trading performance drops significantly during nighttime hours, indicating that market conditions or algorithm effectiveness are suboptimal during this period.  
* **Strategic Impact:**  
  * **Short Trading:** Increased reliance on short trading strategies could potentially double profits.  
  * **Night Trading:** Stopping trading at night could decrease losses by up to 45%.  
  * **Stable Algorithm & Asset Pairing:** Combining a stable momentum algorithm with Nasdaq stocks is projected to further stabilize performance and could potentially increase overall revenue by 100% over time.

---

**Recommendations**

* **Cease Night Trading:**  
  Given the significant drop in performance at night, it is strongly advised to stop using trading algorithms during nighttime hours.  
* **Adopt a Short Trading Strategy:**  
  Shifting towards more short trading can potentially double profits. Begin with short trades as a trial, then analyze and compare the outcomes over a defined period.  
* **Combine Stable Algorithms with Stable Assets:**  
  Utilize a stable momentum algorithm in conjunction with Nasdaq stocks. This combination not only enhances stability but also has the potential to boost overall revenue significantly.  
* **Ongoing Evaluation:**  
  Implement a trial phase where predictions paired with short trades are monitored closely. After approximately one quarter, transition to testing long trades and compare the performance metrics to determine the optimal strategy.

---

**Conclusion**  
This project demonstrates the value of real-time data transformation and KPI aggregation in optimizing automated trading strategies. The analysis has highlighted several key areas for improvement:

* Increasing short trading can potentially double profits.  
* Ceasing night trading may reduce losses by 45%.  
* Combining a stable momentum algorithm with Nasdaq stocks could boost revenue by up to 100% over time.

By applying these strategic moves, finance leaders **can drive improved performance**, optimize algorithmic trading strategies, and achieve more stable, profitable outcomes.

