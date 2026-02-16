# Practicing SQL Case

## Splunk Complex CASE for Bonus Tiers | [prepare.sh](https://prepare.sh/interview/data-analysis/code/complex-case-for-bonus-tiers)

<details>
<summary>See solution</summary>

```sql
WITH bonus_details AS (
    --Show bonus amount and tier for each employee, join with department for the later CTE
    SELECT  e.name,
            e.performance_score,
            e.sales,
            d.id,
            d.department_name,
            d.budget,
    CASE
        WHEN e.sales >= 100000 AND e.performance_score >= 90 THEN 0.2*e.sales
        WHEN e.sales >= 75000 AND e.performance_score >= 80 THEN 0.15*e.sales
        WHEN e.sales >= 50000 AND e.performance_score >= 70 THEN 0.1*e.sales
    ELSE 0.05*e.sales
    END AS bonus_amount,
    CASE
        WHEN e.sales >= 100000 AND e.performance_score >= 90 THEN 'Platinum'
        WHEN e.sales >= 75000 AND e.performance_score >= 80 THEN 'Gold'
        WHEN e.sales >= 50000 AND e.performance_score >= 70 THEN 'Silver'
    ELSE 'Bronze'
    END AS bonus_tier
    FROM employees e
    JOIN departments d
    ON e.department_id = d.id
),
    department_bonus AS (
    --Retrieve each row for each department, the total of bonus and budget for reference in the 3rd CTE
    SELECT  department_name,
            SUM(bonus_amount) AS total_department_bonus,
            budget
    FROM bonus_details
    GROUP BY department_name, budget
),
    ranked_bonus AS (
    --Retrieve each row with bonus ranking by department of each employee, combine employee details with dept total
    SELECT  bd.name,
            bd.sales,
            bd.performance_score,
            bd.department_name,
            bd.bonus_amount,
            bd.bonus_tier,
            RANK() OVER (PARTITION BY bd.department_name ORDER BY bd.bonus_amount DESC) AS department_rank,
            db.total_department_bonus,
            db.budget
    FROM bonus_details bd
    JOIN department_bonus db
    ON bd.department_name = db.department_name
)
    
SELECT  name,
        department_name,
        bonus_tier,
        bonus_amount,
        department_rank,
        total_department_bonus,
        budget,
        CASE
            WHEN total_department_bonus <= budget THEN 'Within Budget'
            ELSE 'Over Budget'
        END AS budget_status
        
FROM ranked_bonus
ORDER BY department_name, department_rank;
```
</details>

-----

## Employee Productivity from Timesheet | [prepare.sh](https://prepare.sh/interview/data-analysis/code/employee-productivity-trend?&technology=SQL&technology=PostgreSQL&difficulty=medium)

<details>
<summary>See solution</summary>
    
```sql
WITH emp_productivity AS (
     SELECT 
            DATE_TRUNC('month', work_date) AS month,
            SUM(hours_worked) AS total_hours,
            SUM(tasks_completed) AS total_tasks,
            employee_id
     FROM timesheets
     GROUP BY employee_id, DATE_TRUNC('month', work_date)
)
SELECT
    TO_CHAR(ep.month, 'YYYY-MM') AS month,
    e.name,
    ROUND(CAST(ep.total_tasks AS DECIMAL)/NULLIF(ep.total_hours,0),2) AS productivity_per_hour
FROM emp_productivity ep
JOIN employees e ON ep.employee_id = e.id
ORDER BY e.name, ep.month;
```

</details>

-----

## Frequent Price Change Detector | [prepare.sh](https://prepare.sh/interview/data-analysis/code/frequent-price-change-detector)

<details>
<summary>See solution</summary>

```sql
WITH cte1 AS (
    SELECT 
        date,
        product_id,
        price,
        LAG(price, 1) OVER (PARTITION BY product_id ORDER BY date ASC) AS last_price
    FROM price_history
),
    cte2 AS (
    SELECT product_id, 
           COUNT(*)::text AS change_count
    FROM cte1
    WHERE price != last_price
        AND last_price IS NOT NULL
    GROUP BY product_id
)


SELECT 
        p.category,
        c.change_count,
        p.name
FROM cte2 c
JOIN products p
ON c.product_id = p.id
WHERE c.change_count::int >= 2
ORDER BY c.change_count::int DESC,
         p.id ASC;
```

</details>

---
## Wayfair YoY Growth Rate | [datalemur](https://datalemur.com/questions/yoy-growth-rate)

Functions: LAG(), CASE WHEN()

<details>
<summary>See solution</summary>
    
```sql

WITH cte1 AS(
  SELECT *,
  EXTRACT(YEAR FROM transaction_date) AS year
  FROM user_transactions
),
cte2 AS (
  SELECT SUM(spend) AS curr_year_spend, 
  product_id, 
  year
  FROM cte1
  GROUP BY product_id, year
  ORDER BY product_id, year
),
cte3 AS (
  SELECT product_id,
  year,
  curr_year_spend,
  LAG(curr_year_spend, 1) OVER (PARTITION BY product_id ORDER BY year ASC) AS prev_year_spend
  FROM cte2
)
SELECT year,
       product_id,
       curr_year_spend,
       prev_year_spend,
       CASE 
        WHEN prev_year_spend IS NOT NULL THEN ROUND(((curr_year_spend/prev_year_spend)-1)*100,2)
        ELSE NULL
       END AS yoy_rate
FROM cte3
ORDER BY product_id, year;

```
</details>
