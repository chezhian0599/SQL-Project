***Credit Card Transactions Analysis***
**Overview**
This project showcases advanced SQL techniques applied to credit card transaction data to extract valuable business insights. 
The repository contains SQL scripts that demonstrate proficiency in SQL functions, window functions, common table expressions (CTEs)
and complex queries, reflecting a deep understanding of data analysis and manipulation.

**Dataset**
The dataset comprises credit card transaction records with attributes such as transaction amount, city, card type, and transaction date. 
The SQL scripts analyze this data to uncover trends and significant metrics related to credit card usage.

**Queries and Analysis**

**1. Top 5 Cities by Credit Card Spends**
Purpose: Identify the top 5 cities with the highest credit card spends and their percentage contribution to the total credit card spends.

SQL Functions Used:

CTE (Common Table Expression): For calculating the total spend.
Aggregation Functions: SUM().
Type Casting: CAST().
Sorting and Limiting Results: ORDER BY and TOP.

```sql
WITH total_spent_cte AS (
    SELECT SUM(CAST(amount AS BIGINT)) AS total_spent 
    FROM credit_card_transcations
)
SELECT TOP 5 city, SUM(amount) AS expense, total_spent,
    CAST((SUM(amount) * 1.0 / total_spent) * 100 AS DECIMAL(5,2)) AS percentage_contribution
FROM credit_card_transcations, total_spent_cte
GROUP BY city, total_spent
ORDER BY expense DESC;

**2. Highest Spend Month by Card Type**
Purpose: Determine the highest spending month for each card type and the amount spent.

SQL Functions Used:

CTE: For pre-aggregating monthly expenses.
Date Functions: DATEPART(), DATENAME().
Window Functions: RANK().

```sql
WITH cte AS (
    SELECT card_type, DATEPART(year, transaction_date) AS yo, DATENAME(month, transaction_date) AS mo,
        SUM(amount) AS monthly_expense
    FROM credit_card_transcations
    GROUP BY card_type, DATEPART(year, transaction_date), DATENAME(month, transaction_date)
)
SELECT * FROM (
    SELECT *, RANK() OVER(PARTITION BY card_type ORDER BY monthly_expense DESC) AS rn
    FROM cte
) A
WHERE rn = 1;


**3. Cumulative Spend Milestones by Card Type**
Purpose: Retrieve transaction details when the cumulative total spends reach 1,000,000 for each card type.

SQL Functions Used:

Window Functions: SUM() OVER(), RANK() OVER().
Conditional Filtering: Using cumulative sums.

```sql
SELECT * FROM (
    SELECT *, RANK() OVER(PARTITION BY card_type ORDER BY cum_sum ASC) AS rn
    FROM (
        SELECT *, SUM(amount) OVER(PARTITION BY card_type ORDER BY transaction_date, transaction_id) AS cum_sum
        FROM credit_card_transcations
    ) A
    WHERE cum_sum >= 1000000
) b
WHERE rn = 1;


**4. City with Lowest Percentage Spend for Gold Card Type**
Purpose: Find the city with the lowest percentage spend for the Gold card type.

SQL Functions Used:

Aggregation Functions: SUM().
Conditional Aggregation: CASE WHEN.
Sorting: ORDER BY.

```sql
SELECT city, SUM(amount) AS total_spend,
    SUM(CASE WHEN card_type = 'Gold' THEN amount ELSE 0 END) AS gold_spend,
    SUM(CASE WHEN card_type = 'Gold' THEN amount ELSE 0 END) * 1.0 / SUM(amount) * 100 AS gold_contribution
FROM credit_card_transcations
GROUP BY city
HAVING SUM(CASE WHEN card_type = 'Gold' THEN amount ELSE 0 END) > 0
ORDER BY gold_contribution;


**Skills Demonstrated**

Proficient use of SQL for data analysis.
Advanced SQL techniques including window functions and common table expressions (CTEs).
Effective use of aggregation, sorting, and filtering to derive insights from data.
