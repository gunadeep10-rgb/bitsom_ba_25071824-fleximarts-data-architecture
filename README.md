# bitsom_ba_25071824-fleximarts-data-architecture
FlexiMart Data Architecture Project
-- Query 1: Customer Purchase History
SELECT
    CONCAT(c.first_name,' ',c.last_name) AS customer_name,
    c.email,
    COUNT(DISTINCT o.order_id) AS total_orders,
    SUM(o.total_amount) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id
HAVING total_orders >= 2 AND total_spent > 5000
ORDER BY total_spent DESC;

-- Query 2: Product Sales Analysis
SELECT
    p.category,
    COUNT(DISTINCT p.product_id) AS num_products,
    SUM(oi.quantity) AS total_quantity_sold,
    SUM(oi.subtotal) AS total_revenue
FROM products p
JOIN order_items oi ON p.product_id = oi.product_id
GROUP BY p.category
HAVING total_revenue > 10000
ORDER BY total_revenue DESC;

-- Query 3: Monthly Sales Trend
SELECT
    MONTHNAME(order_date) AS month_name,
    COUNT(order_id) AS total_orders,
    SUM(total_amount) AS monthly_revenue,
    SUM(SUM(total_amount)) OVER (ORDER BY MONTH(order_date)) AS cumulative_revenue
FROM orders
WHERE YEAR(order_date)=2024
GROUP BY MONTH(order_date)
ORDER BY MONTH(order_date);
