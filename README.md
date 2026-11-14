# **E-commerce SQL Queries**

Explanation of the queries in `simple_project.sql`.

- **Database:** `EcommerceDB` (SQL Server)
- **Tables:** `users`, `products`, `orders`, `events`

---

## **Joins**

### **1. Orders with country and product name**

Joins `orders` with `users` (on `user_id`) and `products` (on `product_id`), so each order shows the customer's country and the product name instead of just IDs.

### **2. Total revenue for each category**

Joins `orders` with `products` to get the category of each order. Then `GROUP BY category` and `SUM(revenue)` give the total per category, sorted from highest to lowest.

### **3. Total revenue for each country**

Same idea as query 2, but it joins `orders` with `users` and groups by `country`.

### **4. Customers who never ordered**

Uses `LEFT JOIN` from `users` to `orders`, which keeps all users even if they have no orders. Users without orders get `NULL` in the order columns, so `WHERE o.order_id IS NULL` returns only the customers who never ordered.

---

## **Window Functions**

A window function does a calculation across related rows **without collapsing them** (unlike `GROUP BY`).

- `PARTITION BY` splits the rows into groups.
- `ORDER BY` inside `OVER` sets the order within each group.

### **5. Rank products inside each category**

An inner query calculates the total revenue for each product. Then `RANK() OVER (PARTITION BY category ORDER BY total_revenue DESC)` ranks the products inside each category, where rank 1 is the highest revenue.

### **6. Monthly revenue with a running total**

The inner query calculates the revenue for each month. Then `SUM(monthly_revenue) OVER (ORDER BY order_month)` adds the months up one after another, so each row shows the total revenue from the first month until that month.

### **7. Monthly revenue compared to the previous month**

`LAG(monthly_revenue) OVER (ORDER BY order_month)` brings the previous month's revenue into the current row. Subtracting it from the current month shows the change from month to month. The first month is `NULL` because it has no previous month.

### **8. First order of each customer**

`ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date, order_id)` numbers each customer's orders from oldest to newest. Keeping only `rn = 1` returns the first order of each customer.
