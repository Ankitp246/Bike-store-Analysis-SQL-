# Bike-store-Analysis-SQL-

# Bike Store SQL Practice
This repository contains SQL queries and data analysis exercises based on a fictional bike store dataset. The dataset includes tables such as customers, orders, products, brands, staff, stores, and more, simulating the operations of a bike store.

# Dataset Overview
The dataset comprises the following tables:

Customers: Information about customers including their names, contact details, and addresses.
Orders: Details of orders placed by customers, including order status, dates, and associated store and staff information.
Products: Information about the products available in the store, including product names, brands, categories, and prices.
Brands: Details about the bike brands sold in the store.
Staff: Information about store staff members, including their roles and contact information.
Stores: Details of the physical stores, including store names, locations, and contact details.
Order Items: Information about items included in each order, such as product IDs, quantities, and prices.
Stocks: Details of product stock levels in each store.

# Importing dataset in postgresql

CREATE TABLE brands(
brand_id INTEGER,	
brand_name VARCHAR
);

COPY brands FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\brands.csv'delimiter ',' csv header;

SELECT * FROM BRANDS;

CREATE TABLE categories(
category_id INTEGER,	
category_name VARCHAR
);

COPY categories FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\categories.csv'DELIMITER ',' csv header;

SELECT * FROM CATEGORIES;

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    email VARCHAR(100),
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(2),
    zip_code VARCHAR(10)
);

COPY customers FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\customers.csv' delimiter ',' csv header;

SELECT * FROM customers;

CREATE TABLE orders_items (
    order_id INTEGER,
    item_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    list_price NUMERIC(10, 2),
    discount NUMERIC(5, 2)
);

COPY orders_items FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\order_items.csv' delimiter ',' csv header;

SELECT * FROM ORDERS;

CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_status VARCHAR(20),
    order_date DATE,
    required_date DATE,
    shipped_date DATE,
    store_id INTEGER,
    staff_id INTEGER
);

COPY orders FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\orders.csv' delimiter ',' csv header;

SELECT * FROM orders;

CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR(255),
    brand_id INTEGER,
    category_id INTEGER,
    model_year INTEGER,
    list_price NUMERIC
);

COPY products FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\products.csv'DELIMITER ',' CSV HEADER;

SELECT * FROM PRODUCTS;

CREATE TABLE staff (
    staff_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20),
    active INTEGER,
    store_id INTEGER,
    manager_id INTEGER
);

COPY staff FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\staffs.csv'DELIMITER ','CSV HEADER;

SELECT * FROM STAFF;

CREATE TABLE stocks(
store_id INTEGER,
product_id INTEGER,
quantity INTEGER
);

COPY stocks FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\stocks.csv'DELIMITER ','CSV HEADER;

SELECT * FROM STOCKS;

CREATE TABLE stores (
    store_id INTEGER PRIMARY KEY,
    store_name VARCHAR(255),
    phone VARCHAR(20),
    email VARCHAR(255),
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(2),
    zip_code VARCHAR(10)
);

COPY stores FROM 'C:\Program Files\PostgreSQL\16\data\data copy\BIKE STORE DATASET\stores.csv' DELIMITER ',' CSV HEADER;

SELECT * FROM STORES;

# Solving Questions 

# Q1: Get the total number of orders in the orders table.

SELECT COUNT(*) AS total_orders 
FROM orders;

# Q2:  Retrieve the total number of customers in each state.

SELECT state, COUNT(customer_id) AS total_customer 
FROM customers
GROUP BY state;

# Q3: Find the top 3 best-selling products based on the total revenue generated.
 
SELECT p.product_id, p.product_name, 
       SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_revenue
FROM products p
JOIN orders_items oi ON p.product_id = oi.product_id
GROUP BY p.product_id, p.product_name
ORDER BY total_revenue DESC
LIMIT 3;

# Q4: Display the order details including customer information (name and email) for each order from the orders table.

SELECT o.order_id,
	o.order_status,
	o.order_date,
	o.shipped_date,
	c.first_name ||' '|| c.last_name AS customer_name,
	c.email
FROM orders AS o JOIN customers AS c 
ON o.customer_id = c.customer_id
ORDER BY o.customer_id;

# Q5: List all products along with their corresponding brand names from the products and brands tables. 

SELECT p.product_name,
	p.brand_id,
	b.brand_name
FROM products AS p JOIN brands AS b 
ON p.brand_id = b.brand_id
ORDER BY p.brand_id;

# Q6: Show the store name, city, and state for each store from the stores table along with the total number of orders placed at each store from the orders table.

SELECT s.store_name,
	s.city,
	s.state,
	s.store_id,
    COUNT(o.*) AS total_orders
FROM stores AS s
LEFT JOIN orders AS o
ON s.store_id = o.store_id
GROUP BY s.store_name, s.city, s.state,s.store_id
ORDER BY s.store_name;

# Q7: Identify the top 5 customers who spent the most on orders.

SELECT c.customer_id, 
       c.first_name || ' ' || c.last_name AS customer_name,
       c.email,
       SUM(oi.quantity * oi.list_price * (1 - oi.discount)) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN orders_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email
ORDER BY total_spent DESC
LIMIT 5;

# Q8: Calculate the total revenue generated by each brand.

SELECT b.brand_id,
	b.brand_name,
	SUM(oi.quantity * oi.list_price *(1-oi.discount)) AS total_revenue
FROM brands AS b 
JOIN products AS p ON b.brand_id = p.brand_id
JOIN orders_items AS oi ON p.product_id = oi.product_id
GROUP BY b.brand_id, b.brand_nameORDER BY total_revenue DESC;
