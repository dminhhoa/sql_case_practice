# sql_case_practice

# Splunk Complex CASE for Bonus Tiers | prepare.sh

Solution

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
        END AS budget_status,
        
FROM ranked_bonus
ORDER BY department_name, department_rank;
```
