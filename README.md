# CRM-Sales-Analysis
## Business Context
This project analyzes the sales operations of a B2B computer hardware firm. In the high-stakes world of hardware—where sales cycles are long and deal values are high—understanding where deals get stuck and who is closing them is the difference between growth and stagnation.
## Tech Stack: SQL (Common Table Expressions, Window Functions, Aggregations)
## SQL SCRIPT FOR COMPLEX QUESTIONS:

####  1. TOTAL REVENUE BY SECTOR 
``` sql
 SELECT 
    sector, 
    ROUND(SUM(revenue), 2) AS total_sector_revenue
FROM 
    accounts
GROUP BY 
    sector
ORDER BY 
    total_sector_revenue DESC;
```

| sector |  total_sector_revenue  |
| :--- | :--- |
| **Software** | 30,950.45 |
| **Technology** | 27,780.53 |
| **Retail** | 27,355.76 |
| **Medical** | 16,976.88 |
| **Telecommunications** | 16,461.91 |
| **Finance** | 16,233.34 |
| **Marketing** | 13,070.38 |
| **Entertainment** | 9,665.22 |
| **Employment** | 6,104.64 |
| **Services** | 4,944.69 |

-  **Top Performers:** Software, Technology, and Retail are the "Big Three," driving nearly 50% of total revenue and showing the strongest product-market fit.

- **Sector Gap:** There is a significant drop-off (over 80%) between the top sector (Software) and the lowest (Services), indicating where sales efforts are most and least efficient.
####  2.Which sales managers have the highest success rate (Win Rate) for their teams
``` sql
SELECT t.manager,COUNT(p.opportunity_id) AS total_deals,
    SUM(CASE WHEN p.deal_stage = 'Won' THEN 1 ELSE 0 END) AS won_deals,
ROUND((SUM(CASE WHEN p.deal_stage = 'Won' THEN 1 ELSE 0 END) * 100.0) / 
        NULLIF(COUNT(p.opportunity_id), 0), 2) AS win_rate_percentage
FROM sales_pipeline p
JOIN sales_teams t ON p.sales_agent = t.sales_agent
GROUP BY t.manager
ORDER BY win_rate_percentage DESC;
```

| manager | total_deals | won_deals | win_rate_percentage |
| :--- | :---: | :---: | :--- |
| **Cara Losch** | 745 | 480 | **64.43** |
| **Summer Sewald** | 1287 | 828 | **64.34** |
| **Celia Rouche** | 962 | 610 | **63.41** |
| **Dustin Brinkmann** | 1186 | 747 | **62.98** |
| **Melvin Marxen** | 1418 | 882 | **62.20** |
| **Rocco Neubert** | 1113 | 691 | **62.08** |

- **Volume vs. Efficiency:** Cara Losch leads in efficiency (64.43%), while Melvin Marxen proves scalability by maintaining a 62.20% win rate across the highest volume of deals (1,418).

- **The Gold Standard:** Summer Sewald represents the "Sweet Spot" of the pipeline, managing over 1,200 deals with a win rate nearly identical to the top spot deals.

#### 3. The elite 5 Sales_agent and managers
``` sql
SELECT 
		t.manager,
    t.regional_office,
    p.sales_agent,
    SUM(p.close_value) AS total_revenue,
    COUNT(p.opportunity_id) AS deals_won,
    ROUND(SUM(CASE WHEN p.deal_stage = 'Won' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 1) AS win_rate_pct
FROM sales_pipeline p
JOIN sales_teams t ON p.sales_agent = t.sales_agent
GROUP BY t.manager, t.regional_office, p.sales_agent
HAVING total_revenue > 0
ORDER BY total_revenue DESC limit 5;
```

| manager | region_office | sales_agent | total_revenue | deals_Won | win_rate_pct |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **Melvin Marxen** | Central | **Darcel Schlecht** | 1,153,214 | 553 | **63.1** |
| **Celia Rouche** | West | **Vicki Laflamme** | 478,396 | 347 | **63.7** |
| **Summer Sewald** | West | **Kary Hendrixson** | 454,298 | 335 | **62.4** |
| **Rocco Neubert** | East | **Cassey Cress** | 450,489 | 261 | **62.5** |
| **Rocco Neubert** | East | **Donn Cantrell** | 445,860 | 275 | **57.5** |

- **Managerial Coaching:** Rocco Neubert is the only manager with two agents in the Top 5, indicating a high-performance culture or a very lucrative territory in the East.

- **Regional Strengths:** The West region shows high efficiency, with both agents maintaining win rates above 62%, while the East shows slightly more variance in conversion (down to 57.5%).

#### 4 What are the top 3 'High-Volume'  products that successfully closed the most deals
``` sql
SELECT product, COUNT(opportunity_id) AS total_deals
FROM sales_pipeline
WHERE deal_stage = 'Won'
GROUP BY product
ORDER BY total_deals DESC limit 3;
```

| product | total_deals |
| :--- | :---: |
| **GTX Basic** | 915 |
| **MG Special** | 793 |
| **GTX Pro** | 729 |

* **Market Leader:** The **GTX Basic** is the clear volume driver with 915 wins, suggesting it is the primary entry point for new customers.
* **Effective Tiering:** The **GTX Pro** (729 wins) shows strong performance, proving that the sales team is successfully upselling customers from the "Basic" model to higher-spec hardware.

 #### 5 What is our product portfolio's pipeline penetration and which high-demand products have the strongest conversion rates?
``` sql
SELECT  p.product,  pr.series, 
    COUNT(p.opportunity_id) AS total_deals_presented, 
    SUM(CASE WHEN p.deal_stage = 'Won' THEN 1 ELSE 0 END) AS total_won, 
    -- Popularity Score (Percentage of total pipeline) 
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM sales_pipeline), 2) AS pct_of_all_deals 
FROM sales_pipeline p 
JOIN products pr ON p.product = pr.product 
GROUP BY p.product, pr.series 
ORDER BY total_deals_presented DESC ;
```

| product | series | total_deals_presented | total_won | pct_of_all_deals |
| :--- | :--- | :---: | :---: | :---: |
| **GTX Basic** | GTX | 1,436 | 915 | **21.40** |
| **MG Special** | MG | 1,223 | 793 | **18.22** |
| **GTX Pro** | GTX | 1,147 | 729 | **17.09** |
| **MG Advanced** | MG | 1,084 | 654 | **16.15** |
| **GTX Plus Basic** | GTX | 1,051 | 653 | **15.66** |
| **GTX Plus Pro** | GTX | 745 | 479 | **11.10** |
| **GTK 500** | GTK | 25 | 15 | **0.37** |


* **GTX Dominance:** The **GTX series** is the company's powerhouse, accounting for over **65%** of all deals presented in the pipeline.
* **Niche Performance:** The **GTK 500** represents a negligible portion (0.37%) of the pipeline, suggesting it is either a highly specialized premium model or a legacy product near the end of its lifecycle.
* **Volume Efficiency:** Despite the high volume of **GTX Basic**, it maintains a high win rate, proving that popularity is backed by strong customer demand and sales execution.


####  6.Which products act as 'Time Sinks' by consuming excessive  sales effort on lost opportunities relative to successful conversions
``` sql
SELECT p.product,pr.series,
    COUNT(CASE WHEN p.deal_stage = 'Won' THEN 1 END) AS won_count,
    ROUND(AVG(CASE WHEN p.deal_stage = 'Won' THEN DATEDIFF(p.close_date, p.engage_date) END), 0) AS avg_days_to_win,
    COUNT(CASE WHEN p.deal_stage = 'Lost' THEN 1 END) AS lost_count,
    ROUND(AVG(CASE WHEN p.deal_stage = 'Lost' THEN DATEDIFF(p.close_date, p.engage_date) END), 0) AS avg_days_to_lose,
    ROUND(SUM(CASE WHEN p.deal_stage = 'Won' THEN p.close_value ELSE 0 END) / 
          NULLIF(SUM(DATEDIFF(p.close_date, p.engage_date)), 0), 2) AS revenue_per_engagement_day
FROM sales_pipeline p
JOIN products pr ON p.product = pr.product
WHERE p.deal_stage IN ('Won', 'Lost')
GROUP BY p.product, pr.series

```
| product | series | won_count | avg_days_to_win | lost_count | avg_days_to_lose | revenue_per_engagement_day |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **GTX Pro** | GTX | 729 | 48 | 418 | 41 | **66.92** |
| **MG Special** | MG | 793 | 51 | 430 | 43 | **0.74** |
| **GTX Plus Basic** | GTX | 653 | 52 | 398 | 46 | **13.58** |
| **GTX Plus Pro** | GTX | 479 | 52 | 266 | 36 | **76.61** |
| **MG Advanced** | MG | 654 | 52 | 430 | 40 | **43.39** |
| **GTX Basic** | GTX | 915 | 55 | 521 | 41 | **6.97** |
| **GTK 500** | GTK | 15 | 64 | 10 | 38 | **298.30** |

* **The Velocity King:** **GTX Pro** has the shortest sales cycle, closing in just **48 days**. This represents the most efficient path from engagement to revenue.
* **High Stakes / Long Cycle:** **GTK 500** takes the longest to win (**64 days**). However, it produces a massive **$298.30 in revenue per engagement day**, suggesting that while it is slow, the payout is significantly higher per hour worked.

#### 7.Which products have the most consistent sales price,  and which ones suffer from high price volatility?
``` sql
SELECT product, 
    MIN(close_value) AS lowest_sale,
    MAX(close_value) AS highest_sale,
    AVG(close_value) AS average_sale,
    COUNT(opportunity_id) AS total_sold
FROM sales_pipeline
WHERE deal_stage = 'Won'
GROUP BY product
ORDER BY average_sale DESC;
```

| product | lowest__sale | highest_sale | average_sale | total_sold |
| :--- | :--- | :--- | :--- | :---: |
| **GTK 500** | 23,746 | 30,288 | **26,707.47** | 15 |
| **GTX Plus Pro** | 3,704 | 7,356 | **5,489.88** | 479 |
| **GTX Pro** | 3,169 | 6,166 | **4,815.61** | 729 |
| **MG Advanced** | 2,477 | 4,456 | **3,388.97** | 654 |
| **GTX Plus Basic** | 752 | 1,374 | **1,080.05** | 653 |
| **GTX Basic** | 365 | 716 | **545.64** | 915 |
| **MG Special** | 38 | 72 | **55.19** | 793 |

* **The "Whale" Product:** The **GTK 500** is in a category of its own. With an average sale of **26,707**, a single "Won" deal for this product is worth more than 793 total sold  of the **MG Special**. 
* **Pricing Segmentation:** There is a clear "Luxury" gap between the **GTX Pro** ($4,815 avg) and the **GTX Plus Basic** ($1,080 avg). This suggests the "Pro" designation carries a significant price premium that customers are willing to pay.


####  8.How did sales pipeline activity  and conversion success trend across the four quarters of 2017?"
```  sql
SELECT YEAR(close_date) AS sales_year,
    QUARTER(close_date) AS sales_quarter,
    SUM(close_value) AS total_revenue,
    COUNT(opportunity_id) AS deals_won
FROM sales_pipeline
WHERE deal_stage = 'Won'
GROUP BY sales_year, sales_quarter
ORDER BY total_revenue DESC;
```

| sales_year | sales_quarter | total_revenue | deals_won |
| :--- | :---: | :--- | :---: |
| 2017 | **2** | **3,086,111** | 1,254 |
| 2017 | **3** | 2,982,255 | 1,257 |
| 2017 | **4** | 2,802,496 | 1,196 |
| 2017 | **1** | 1,134,672 | 531 |


* **The Q2 Surge:** Revenue exploded in the second quarter, nearly **tripling** the performance of Q1. This indicates a massive shift in market traction or a highly successful seasonal campaign.
* **Volume vs. Value:** Interestingly, Q3 saw the highest number of **deals won (1,257)**, but Q2 generated the **highest revenue**. This suggests that Q2 focused on high-value "Enterprise" contracts, while Q3 was driven by a higher volume of smaller deals.

####  9."What is each account's contribution to our total success, and what percentage of our total won deals does each account represent?"
``` sql
SELECT account,
    COUNT(opportunity_id) AS total_deals_won,
    ROUND(COUNT(opportunity_id) * 100.0 / (SELECT COUNT(*) FROM sales_pipeline WHERE deal_stage = 'Won'), 2) AS pct_of_total_wins,
    SUM(close_value) AS total_revenue_generated,
    ROUND(AVG(DATEDIFF(close_date, engage_date)), 0) AS avg_sales_cycle_days
FROM sales_pipeline
WHERE deal_stage = 'Won'
GROUP BY account
ORDER BY total_revenue_generated DESC limit 5;
```

| account | total_deals_won |  pct_of_total_wins|total_revenue_generated |avg_sales_cycle_days |
| :--- | :---: | :---: | :--- | :---: |
| **Kan-code** | 115 | 2.71 | **341,455** | 51 |
| **Konex** | 108 | 2.55 | **269,245** | 48 |
| **Condax** | 105 | 2.48 | **206,410** | 58 |
| **Cheers** | 57 | 1.34 | **198,020** | 48 |
| **Hottechi** | 111 | 2.62 | **194,957** | 56 |


* **The "Whale" Client:** **Kan-code** is the primary revenue driver, contributing **$341,455** across 115 deals.
* **High-Value Efficiency:** **Cheers** generates significant revenue ($198k) with only 57 deals, suggesting a much higher average deal size compared to the other top accounts.
* **Velocity Leaders:** **Konex** and **Cheers** boast the fastest turnaround time at **48 days**, significantly outperforming the 58-day cycle seen with **Condax**.


####  10.What is the financial weight of each account, and which clients  contribute the largest percentage to the company's total bottom line?
``` sql
SELECT  account,
    SUM(close_value) AS total_revenue_generated,
    -- Calculating the % of total revenue
    ROUND(SUM(close_value) * 100.0 / (SELECT SUM(close_value) FROM sales_pipeline WHERE deal_stage = 'Won'), 2) AS pct_of_total_revenue,
    COUNT(opportunity_id) AS total_deals_won,
    ROUND(AVG(DATEDIFF(close_date, engage_date)), 0) AS avg_sales_cycle_days
FROM sales_pipeline
WHERE deal_stage = 'Won'
GROUP BY account
ORDER BY total_revenue_generated DESC limit 5;
```

| Account | Total Revenue ($) | % of Total Revenue | Total Deals Won | Avg Sales Cycle (Days) |
| :--- | :--- | :---: | :---: | :---: |
| **Kan-code** | **341,455** | **3.41** | 115 | 51 |
| **Konex** | **269,245** | **2.69** | 108 | 48 |
| **Condax** | **206,410** | **2.06** | 105 | 58 |
| **Cheers** | **198,020** | **1.98** | 57 | 48 |
| **Hottechi** | **194,957** | **1.95** | 111 | 56 |

* **The Revenue Anchor:** **Kan-code** is the company's most significant financial partner, responsible for **3.41%** of the total bottom line.
* **Low Risk Exposure:** Because the top client represents less than **4%** of total revenue, the company's financial health is well-diversified and not overly dependent on any single contract.
* **Cycle Comparison:** **Konex** and **Cheers** share the fastest turnaround time (**48 days**), making them the most "liquid" high-value accounts in the portfolio.


## 🎯 Final Strategic Recommendations

Based on the comprehensive analysis of the 2017 sales pipeline (CRM Anyalsis), the following strategic actions are recommended to optimize revenue and operational efficiency:



### 1. Protect Your Time (The "Fail Fast" Rule)
* **Recommendation:** Set a **40-day "Hard Stop"** for the premium GTK 500 series.
* ** Explanation:** "Our data shows it takes 64 days to win a big deal but only 38 days to lose one. If a deal isn't moving by day 40, we should drop it. This prevents 'Time Sinks' and lets sales reps focus on leads that actually close."

### 2. Double Down on the "Sweet Spot"
* **Recommendation:** Move more marketing budget toward the **GTX Pro**.
* **Explanation:** "The GTX Pro is our most efficient product—it has the fastest sales cycle (48 days) and high volume. It’s the perfect balance of speed and revenue. Selling more of these is the fastest way to grow."

### 3. Clone Your Best Customers
* **Recommendation:** Create a 'VIP Profile' based on the **Cheers** account.
* **Explanation:** "The account 'Cheers' generates huge revenue with very few deals. Instead of chasing 100 small customers, we should find 10 more customers that look exactly like 'Cheers' to maximize our profit per hour."

### 4. Scale the "West" Success
* **Recommendation:** Move top-performing tactics from the **West Region** to the East and Central.
* **Explanation:** "The West is our strongest region in both revenue and win rate. We should have our West Coast managers, like Rocco Neuman, train the other regions on their specific sales scripts and closing techniques."
























