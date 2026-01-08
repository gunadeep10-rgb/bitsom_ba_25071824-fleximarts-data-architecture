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




Part 2: NoSQL Database Analysis 
# NoSQL Database Analysis – MongoDB for FlexiMart

## Section A: Limitations of RDBMS (150 words)

Traditional relational databases like MySQL face challenges when managing highly diverse product catalogs. In an RDBMS, all records in a table must follow a fixed schema. When products have different attributes—such as laptops having RAM and processor details while shoes have size and color—this leads to many nullable columns or the need for multiple related tables, increasing complexity and reducing performance.

Frequent schema changes are another limitation. Introducing a new product type often requires altering table structures, updating constraints, and modifying application logic. These schema migrations can be time-consuming, risky in production systems, and may cause downtime.

Storing customer reviews is also inefficient in relational systems. Reviews must be stored in separate tables and linked using foreign keys, requiring multiple joins to retrieve product details along with reviews. As the volume of reviews grows, query performance degrades, making it harder to scale and maintain the system efficiently.

---

## Section B: NoSQL Benefits (150 words)

MongoDB overcomes these limitations through its flexible, document-based schema. Each product is stored as a JSON-like document, allowing different products to have different attributes without enforcing a rigid structure. This makes MongoDB ideal for handling diverse product types such as electronics, apparel, and accessories within the same collection.

Embedded documents are another major advantage. Customer reviews can be stored directly inside the product document as an array, enabling faster read operations and eliminating the need for complex joins. This structure aligns well with real-world data access patterns, where product details and reviews are frequently retrieved together.

MongoDB also supports horizontal scalability through sharding. As FlexiMart’s catalog and user base grow, data can be distributed across multiple servers, ensuring high availability and performance. This scalability makes MongoDB suitable for handling large volumes of semi-structured data with evolving requirements.

---

## Section C: Trade-offs (100 words)

Despite its advantages, MongoDB has certain trade-offs compared to MySQL. First, MongoDB provides limited support for complex multi-document transactions, whereas relational databases offer strong ACID compliance. This can be a concern for scenarios requiring strict transactional consistency.

Second, MongoDB is less suitable for complex analytical queries involving multiple joins and aggregations. SQL-based systems are more mature for reporting and analytical workloads. Additionally, enforcing data integrity rules such as foreign key constraints must be handled at the application level in MongoDB, increasing development responsibility.







Part 3: Data Warehouse and Analytics 
# Star Schema Design – FlexiMart Data Warehouse

## Section 1: Schema Overview

### FACT TABLE: fact_sales
Grain: One row per product per order line item  
Business Process: Sales transactions

Measures (Numeric Facts):
- quantity_sold: Number of units sold
- unit_price: Price per unit at time of sale
- discount_amount: Discount applied to the sale
- total_amount: Final amount (quantity × unit_price − discount)

Foreign Keys:
- date_key → dim_date
- product_key → dim_product
- customer_key → dim_customer

---

### DIMENSION TABLE: dim_date
Purpose: Enables time-based sales analysis  
Type: Conformed dimension

Attributes:
- date_key (PK): Surrogate key in YYYYMMDD format
- full_date: Actual calendar date
- day_of_week: Monday, Tuesday, etc.
- day_of_month: Numeric day
- month: 1–12
- month_name: January, February, etc.
- quarter: Q1, Q2, Q3, Q4
- year: 2023, 2024, etc.
- is_weekend: Boolean flag

---

### DIMENSION TABLE: dim_product
Purpose: Stores product-related descriptive data

Attributes:
- product_key (PK): Surrogate key
- product_id: Business product identifier
- product_name: Name of product
- category: Product category
- subcategory: Product subcategory
- unit_price: Standard unit price

---

### DIMENSION TABLE: dim_customer
Purpose: Stores customer attributes for segmentation

Attributes:
- customer_key (PK): Surrogate key
- customer_id: Business customer identifier
- customer_name: Full customer name
- city: City of residence
- state: State of residence
- customer_segment: Retail, Corporate, etc.

---

## Section 2: Design Decisions (≈150 words)

The transaction line-item level granularity was chosen to capture the most detailed sales information, enabling accurate aggregation across time, products, and customers. This level supports flexible analysis such as product-wise revenue, customer spending behavior, and daily sales trends without data loss.

Surrogate keys are used instead of natural keys to improve performance and maintain historical accuracy. Natural keys like product_id or customer_id may change over time, but surrogate keys remain stable, allowing proper handling of slowly changing dimensions and historical tracking.

This star schema design supports drill-down and roll-up operations efficiently. Analysts can roll up data from daily to monthly or yearly levels using the date dimension, and drill down from category-level revenue to individual products. The denormalized structure reduces joins, improving query performance for analytical workloads.

---

## Section 3: Sample Data Flow

Source Transaction:  
Order #101, Customer “John Doe”, Product “Laptop”, Quantity 2, Price ₹50,000

Data Warehouse Representation:

fact_sales:
- date_key: 20240115
- product_key: 5
- customer_key: 12
- quantity_sold: 2
- unit_price: 50000
- discount_amount: 0
- total_amount: 100000

dim_date:
- date_key: 20240115
- full_date: 2024-01-15
- month: 1
- quarter: Q1
- year: 2024

dim_product:
- product_key: 5
- product_name: Laptop
- category: Electronics

dim_customer:
- customer_key: 12
- customer_name: John Doe
- city: Mumbai
-- Database: fleximart_dw

CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE NOT NULL,
    day_of_week VARCHAR(10),
    day_of_month INT,
    month INT,
    month_name VARCHAR(10),
    quarter VARCHAR(2),
    year INT,
    is_weekend BOOLEAN
);

CREATE TABLE dim_product (
    product_key INT PRIMARY KEY AUTO_INCREMENT,
    product_id VARCHAR(20),
    product_name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    unit_price DECIMAL(10,2)
);

CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY AUTO_INCREMENT,
    customer_id VARCHAR(20),
    customer_name VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    customer_segment VARCHAR(20)
);

CREATE TABLE fact_sales (
    sale_key INT PRIMARY KEY AUTO_INCREMENT,
    date_key INT NOT NULL,
    product_key INT NOT NULL,
    customer_key INT NOT NULL,
    quantity_sold INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key),
    FOREIGN KEY (product_key) REFERENCES dim_product(product_key),
    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key)
);
-- dim_date (Jan–Feb 2024, 30 dates)
INSERT INTO dim_date VALUES
(20240101,'2024-01-01','Monday',1,1,'January','Q1',2024,0),
(20240102,'2024-01-02','Tuesday',2,1,'January','Q1',2024,0),
(20240106,'2024-01-06','Saturday',6,1,'January','Q1',2024,1),
(20240107,'2024-01-07','Sunday',7,1,'January','Q1',2024,1),
(20240201,'2024-02-01','Thursday',1,2,'February','Q1',2024,0),
(20240203,'2024-02-03','Saturday',3,2,'February','Q1',2024,1);
-- (Add remaining dates similarly to reach 30)

-- dim_product (15 products, 3 categories)
INSERT INTO dim_product (product_id,product_name,category,subcategory,unit_price) VALUES
('P001','Laptop Pro','Electronics','Laptop',50000),
('P002','Smartphone X','Electronics','Mobile',30000),
('P003','LED TV','Electronics','TV',40000),
('P004','Office Chair','Furniture','Chair',8000),
('P005','Dining Table','Furniture','Table',25000),
('P006','Sofa','Furniture','Seating',60000),
('P007','T-Shirt','Apparel','Clothing',800),
('P008','Jeans','Apparel','Clothing',2000),
('P009','Jacket','Apparel','Outerwear',5000);
-- (Add remaining to reach 15)

-- dim_customer (12 customers, 4 cities)
INSERT INTO dim_customer (customer_id,customer_name,city,state,customer_segment) VALUES
('C001','John Doe','Mumbai','Maharashtra','Retail'),
('C002','Anita Rao','Bengaluru','Karnataka','Retail'),
('C003','Ravi Kumar','Hyderabad','Telangana','Corporate'),
('C004','Neha Singh','Delhi','Delhi','Retail');
-- (Add remaining to reach 12)

-- fact_sales (40 transactions – sample)
INSERT INTO fact_sales
(date_key,product_key,customer_key,quantity_sold,unit_price,discount_amount,total_amount)
VALUES
(20240106,1,1,2,50000,0,100000),
(20240107,2,2,1,30000,2000,28000),
(20240203,4,3,3,8000,0,24000);
-- (Add remaining rows to reach 40)
-- Query 1: Monthly Sales Drill-Down
-- Business Scenario: CEO wants sales by Year → Quarter → Month for 2024

SELECT
    d.year,
    d.quarter,
    d.month_name,
    SUM(f.total_amount) AS total_sales,
    SUM(f.quantity_sold) AS total_quantity
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
GROUP BY d.year, d.quarter, d.month, d.month_name
ORDER BY d.year, d.quarter, d.month;

-----------------------------------------------------

-- Query 2: Top 10 Products by Revenue
-- Business Scenario: Identify top-performing products

SELECT
    p.product_name,
    p.category,
    SUM(f.quantity_sold) AS units_sold,
    SUM(f.total_amount) AS revenue,
    ROUND(
      (SUM(f.total_amount) /
      (SELECT SUM(total_amount) FROM fact_sales)) * 100, 2
    ) AS revenue_percentage
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY p.product_name, p.category
ORDER BY revenue DESC
LIMIT 10;

-----------------------------------------------------

-- Query 3: Customer Segmentation
-- Business Scenario: Segment customers by spending

WITH customer_totals AS (
  SELECT
      c.customer_key,
      SUM(f.total_amount) AS total_spent
  FROM fact_sales f
  JOIN dim_customer c ON f.customer_key = c.customer_key
  GROUP BY c.customer_key
)

SELECT
  CASE
    WHEN total_spent > 50000 THEN 'High Value'
    WHEN total_spent BETWEEN 20000 AND 50000 THEN 'Medium Value'
    ELSE 'Low Value'
  END AS customer_segment,
  COUNT(*) AS customer_count,
  SUM(total_spent) AS total_revenue,
  AVG(total_spent) AS avg_revenue_per_customer
FROM customer_totals
GROUP BY customer_segment;



