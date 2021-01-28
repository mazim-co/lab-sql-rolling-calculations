-- LAB SQL Rolling calculations
-- Get number of monthly active customers.
CREATE OR REPLACE VIEW user_activity AS
SELECT customer_id, CONVERT(rental_date, DATE) AS Activity_date,
date_format(rental_date, '%m') AS Activity_Month,
date_format(CONVERT(rental_date,DATE), '%Y') AS Activity_year
FROM sakila.rental;

CREATE OR REPLACE VIEW monthly_activity AS
SELECT count(DISTINCT customer_id) AS Active_users, Activity_year, Activity_Month
FROM user_activity
GROUP BY Activity_year, Activity_Month
ORDER BY Activity_year, Activity_Month;

-- Active users in the previous month.

WITH cte_activity AS (
  SELECT Active_users, lag(Active_users,1) over (PARTITION BY Activity_year) AS last_month, Activity_year, Activity_month
  FROM Monthly_activity
)
SELECT * FROM cte_activity
WHERE last_month IS NOT NULL;


-- Percentage change in the number of active customers.

WITH cte_activity AS (
  SELECT Active_users, lag(Active_users,1) over (PARTITION BY Activity_year) AS last_month, Activity_year, Activity_month
  FROM Monthly_activity
)
SELECT *, round(((Active_users-last_month)/last_month)*100) AS percentage FROM cte_activity
WHERE last_month IS NOT NULL;

-- Retained customers every month.

WITH distinct_users AS (
  SELECT DISTINCT customer_id , Activity_month, Activity_year 
  FROM user_activity
)
SELECT count(DISTINCT d1.customer_id) AS Retained_customers, d1.Activity_month, d1.Activity_year
FROM distinct_users d1
JOIN distinct_users d2
ON d1.customer_id = d2.customer_id AND d1.activity_month = d2.activity_month + 1
GROUP BY d1.Activity_month, d1.Activity_year
ORDER BY d1.Activity_year, d1.Activity_month;


