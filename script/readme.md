-- ========== JOINS ==========

-- 1) Show orders with the customer's country and the product name
SELECT TOP 10 o.order_id, u.country, p.product_name, o.revenue
FROM orders o
JOIN users u ON o.user_id = u.user_id
JOIN products p ON o.product_id = p.product_id;

-- 2) Total revenue for each category
SELECT p.category, SUM(o.revenue) AS total_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.category
ORDER BY total_revenue DESC;

-- 3) Total revenue for each country
SELECT u.country, SUM(o.revenue) AS total_revenue
FROM orders o
JOIN users u ON o.user_id = u.user_id
GROUP BY u.country
ORDER BY total_revenue DESC;

-- 4) Customers who signed up but never ordered
SELECT u.user_id, u.country
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id IS NULL;

-- ========== WINDOW FUNCTIONS ==========

-- 5) Rank products by revenue inside each category
SELECT category, product_name, total_revenue,
       RANK() OVER (PARTITION BY category ORDER BY total_revenue DESC) AS rank_in_category
FROM (
    SELECT p.category, p.product_name, SUM(o.revenue) AS total_revenue
    FROM orders o
    JOIN products p ON o.product_id = p.product_id
    GROUP BY p.category, p.product_name
) t;

-- 6) Monthly revenue with a running total
SELECT order_month, monthly_revenue,
       SUM(monthly_revenue) OVER (ORDER BY order_month) AS running_total
FROM (
    SELECT CONVERT(VARCHAR(7), order_date, 120) AS order_month, SUM(revenue) AS monthly_revenue
    FROM orders
    GROUP BY CONVERT(VARCHAR(7), order_date, 120)
) t;

-- 7) Monthly revenue compared to the previous month
SELECT order_month, monthly_revenue,
       LAG(monthly_revenue) OVER (ORDER BY order_month) AS previous_month,
       monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY order_month) AS difference
FROM (
    SELECT CONVERT(VARCHAR(7), order_date, 120) AS order_month, SUM(revenue) AS monthly_revenue
    FROM orders
    GROUP BY CONVERT(VARCHAR(7), order_date, 120)
) t;

-- 8) First order of each customer
SELECT user_id, order_id, order_date, revenue
FROM (
    SELECT user_id, order_id, order_date, revenue,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date, order_id) AS rn
    FROM orders
) t
WHERE rn = 1;
