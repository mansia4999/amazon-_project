# amazon-_project


CREATE DATABASE applestore;



-- creating store table

DROP TABLE IF EXISTS stores;
CREATE TABLE stores
    (
        store_id INT PRIMARY KEY, 	
        store_name VARCHAR(55),	
        country	VARCHAR(25),
        city VARCHAR(55)
    );




DROP TABLE IF EXISTS products;
CREATE TABLE products
    (
        product_id	INT PRIMARY KEY,
        product_name	VARCHAR(55),
        category	VARCHAR(25),
        price	FLOAT,
        launched_price FLOAT,	
        cogs FLOAT
    )

SELECT * FROM products;


ALTER TABLE products
ADD CONSTRAINT pk_products
PRIMARY KEY (product_id);


DROP TABLE IF EXISTS sales;
CREATE TABLE sales
    (
    sale_id INT PRIMARY KEY,
    store_id INT, --- fk
    product_id INT, --- fk
    sale_date DATE,
    quantity INT,
    CONSTRAINT FK_stores FOREIGN KEY (store_id) REFERENCES stores(store_id)
    );


select * from sales;

ALTER TABLE sales
ADD CONSTRAINT fk_products
FOREIGN KEY (product_id)
REFERENCES products(product_id);



-- Day2 continue...


SELECT * FROM products;
SELECT * FROM stores;
SELECT * FROM sales;

INSERT INTO products(product_id, product_name, launched_price)
VALUES
    (31, 'iPhone 16', 1299);


DROP TABLE IF EXISTS sales;


-- DML Function
-- Feature engineering
-- CTAS


SELECT * FROM stores;

-- Add records into stores

INSERT INTO stores
VALUES
(56, 'Apple Store Pune', 'India', 'Pune');

INSERT INTO stores(store_id, store_name)
VALUES
(57, 'Apple Store Gurgaon'),
(58, 'Apple Store 2');

UPDATE stores
SET country = 'India'
WHERE store_id = 57;

SELECT * FROM stores;


UPDATE stores
SET country = 'India',
    city = 'Gurgaon'
WHERE store_id = 58;



-- TRUNCATE vs DELETE
-- DML
-- DDL

TRUNCATE TABLE stores;



DROP TABLE IF EXISTS sales;
CREATE TABLE sales
    (
    sale_id INT PRIMARY KEY,
    store_id INT, --- fk
    product_id INT, --- fk
    sale_date DATE,
    quantity INT,
    CONSTRAINT FK_stores FOREIGN KEY (store_id) REFERENCES stores(store_id),
    CONSTRAINT fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
    );


select * from products;
select * from stores;
select * from sales;


ALTER TABLE sales
ADD COLUMN sale_price FLOAT;


ALTER TABLE sales
ADD COLUMN day_of_sale VARCHAR(10);



SELECT TO_CHAR(CURRENT_DATE, 'Day')


SELECT 
    *,
    LENGTH(TRIM(TO_CHAR(sale_date, 'day')))
FROM sales;




UPDATE sales
SET day_of_sale = TRIM(TO_CHAR(sale_date, 'day'));


select * from sales;


update sales as s
set	sale_price = p.price * s.quantity
from products as p
where s.product_id = p.product_id


-- total sales from india in last 5 years

select  
	extract(year from s.sale_date)as years,
	extract(month from s.sale_date)as months,
	sum(s.sale_price) as revenue,
	count(*)
from sales as s
 join 
stores as st
	on s.store_id = st.store_id
where st.country ='India'
and
s.sale_date >= current_date - interval '5 years'
group by 1,2
order by 1,3 desc

-- select current_date - interval '5 years'

-- Get me last 3 years sale, total qty, total orders, 
-- for each country

--  here using CTAS
create table latest_3_years_reports
	as
SELECT 
        st.country,
        -- EXTRACT(YEAR FROM s.sale_date) as years,
        -- EXTRACT(MONTH FROM s.sale_date) as month,
        SUM(s.sale_price) as revenue,
        COUNT(*) as total_orders,
        SUM(s.quantity) as unit_sold
FROM sales as s
JOIN
stores as st
ON s.store_id = st.store_id
WHERE s.sale_date >= CURRENT_DATE - INTERVAL '3 years'
GROUP BY 1
ORDER BY 1, 4 DESC;

select * from latest_3_years_reports



-- interview questions!!

select * from products;        /* product_id,product_name,cattegory,price,launched_price,cogs*/
select * from stores;    /* store_id,store_name,country,city*/
select * from  sales;    /* sale_id,store_id,product_id,sale_date,quantity,sale_prices,day_of_sale */



/*
Find top 5 product with highest profit 
Total Sales by Product: How many units of each product have been sold?
Sales by Store: How many units have been sold in each store?
Top-Selling Products: What are the top 5 best-selling products by quantity?
Top Revenue-Generating Stores: Which are the top 5 stores generating the highest revenue?

*/



--  need to practice above ... ques

-- CASE STATEMENT


-- Classifying Products by Price Range: Classify products into different price ranges: 'Budget', 'Mid-Range', and 'Premium'
    
-- Sales Performance Evaluation: Evaluate the sales performance of each store based on total sales quantity: 'Low', 'High'. (if greater than average of sale call high else call low

-- Classifying Products by Price Range: Classify products into different price ranges: 'Budget', 'Mid-Range', and 'Premium'
    
-- budget < 500
-- mid- 500 and 1000
-- Premium > 1000



SELECT 
    prod_category,
    COUNT(*) as cnt_product
FROM 
(SELECT 
    *,
    CASE
        WHEN price < 500 THEN 'budget'
        WHEN price BETWEEN 500 AND 1000 THEN 'mid_range'
        ELSE 'premium'
    END as prod_category
FROM products
) t2
GROUP BY 1






32443.54545454

SELECT ROUND(32443.54545454, 2)

SELECT ABS(-5)

SELECT RANDOM()

SELECT CONCAT('john', ' ', 'smith')
    
SELECT LENGTH('Hello world')

SELECT LEFT('Hello world', 2)

-- Wildcard

SELECT * FROM products
WHERE product_name LIKE '__r%'


SELECT * FROM products
WHERE product_name LIKE 'i%x'

--  usage of CTAS


-- 
CREATE TABLE orders_summary
AS    
SELECT 
        s.sale_id,
        s.sale_date,
        p.product_id,
        p.product_name,
        s.quantity,
        s.sale_price,
        p.cogs,
        st.store_id,
        st.store_name,
        s.day_of_sale,
        st.country
FROM products as p
JOIN 
sales as s
    ON s.product_id = p.product_id
JOIN stores as st
ON s.store_id = st.store_id


	select * from orders_summary


	
EXPLAIN ANALYZE -- et 408.0 ms pt -- .45 ms
SELECT * FROM 
orders_summary


--  here the concept of index and explain analyze as well

CREATE INDEX country_index ON orders_summary(country);

EXPLAIN ANALYZE -- 724 ms
SELECT * FROM 
orders_summary
WHERE sale_date = '2017-10-02'


CREATE INDEX date_index ON orders_summary(sale_date);


--  pros of index : query is fastly performed and it is used for Read purpose only
--  cons: now update will take  more time

--  View

-- VIEW 
DROP VIEW IF EXISTS orders_view;
CREATE VIEW orders_view
AS    
SELECT 
        s.sale_id,
        s.sale_date,
        p.product_id,
        -- p.product_name,
        s.quantity,
        s.sale_price,
        -- p.cogs,
        -- st.store_id,
        st.store_name,
        -- s.day_of_sale,
        st.country as countri
FROM products as p
JOIN 
sales as s
    ON s.product_id = p.product_id
JOIN stores as st
ON s.store_id = st.store_id
WHERE s.sale_date > CURRENT_DATE - INTERVAL '5 year'

select * from orders_view;




SELECT 
    store_name,
    EXTRACT(YEAR FROM sale_date) as year,
    SUM(sale_price) as revenue
FROM orders_view
WHERE countri = 'UK'
GROUP BY 1, 2
ORDER BY 1, 3 DESC





-- Window Functions

CTAS, VIEWS, INDEX

-- windows function!!

-- Store Performance Analysis: How are stores ranked based on their performance?

	    
SELECT 
    store_name,
    COUNT(*) as total_orders,
    RANK() OVER(ORDER BY COUNT(*) DESC) as rank,
    ROW_NUMBER() OVER(ORDER BY COUNT(*) DESC) as rn
FROM orders_summary
GROUP BY 1

    
Find the product with highest total profit from each country of 2020 (return country, product name, total unit sold, total profit
Find out top 5 store with the highest sale



--  here we are usig window unct.. without using partion(global ranking) with using partion by(local ranking) i.e is kind of group!!


DROP TABLE IF EXISTS ordders;
CREATE TABLE ordders(id SERIAL, sale_amt INT, store_name VARCHAR(5));

INSERT INTO ordders(sale_amt, store_name)
VALUES
(55, 'A'),
(50, 'A'),
(25, 'A'),
(25, 'A'),
(18, 'A'),
(65, 'B'),
(65, 'B'),
(50, 'B'),
(15, 'B'),
(18, 'C'),    
(16, 'C') ,  
(16, 'C')  ,  
(13, 'C'),
(13, 'C'),
(11, 'C');
SELECT * FROM ordders;

-- 3rd highest, highest, 5th highest orders amt details
-- GLOBAL RANKING
--  here noraml wf
SELECT *,
    ROW_NUMBER() OVER(ORDER BY sale_amt DESC) as row_number,
    RANK() OVER(ORDER BY sale_amt DESC) as rank,
    DENSE_RANK() OVER(ORDER BY sale_amt DESC) as d_rank
FROM ordders;


-- 3rd highest salary
-- 3rd highest amt order details
-- LOCAL RANKING
SELECT 
    id,
    sale_amt,
    store_name
FROM

(SELECT *,
    -- ROW_NUMBER() OVER(ORDdER BY sale_amt DESC) as row_number,
    -- RANK() OVER(ORDdER BY sale_amt DESC) as rank,
    DENSE_RANK() OVER(PARTITION BY store_name order BY sale_amt DESC) as d_rank
FROM ordders
) t1
WHERE d_rank = 3

GLOBAL RANKING
LOCAL RANKING
-- PARTITION BY 

-- For each store we want to see 3nd highest sale_amt order details

SELECT * FROM ordders;




-- Amazon Interview Practice 21/02/2024


/* 1. You have two tables: Product and Supplier.
- Product Table Columns: Product_id, Product_Name, Supplier_id, Price
- Supplier Table Columns: Supplier_id, Supplier_Name, Country
*/



-- Write an SQL query to find the name of the product with the highest 
-- price in each country.


-- creating supplier table 
DROP TABLE IF EXISTS suppliers;
CREATE TABLE suppliers(supplier_id int PRIMARY KEY,
					  supplier_name varchar(25),
					  country VARCHAR(25)
					  );
-- let's insert some values 

INSERT INTO suppliers
VALUES(501, 'alan', 'India'),
		(502, 'rex', 'US'),
		(503, 'dodo', 'India'),
		(504, 'rahul', 'US'),
		(505, 'zara', 'Canda'),
		(506, 'max', 'Canada')
;

select * from suppliers;

-- creating the productts table 

DROP TABLE IF EXISTS productts;
CREATE TABLE  productts(

						product_id int PRIMARY KEY,
						product_name VARCHAR(25),
						supplier_id int,
						price float,
						FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
						);

INSERT INTO  productts
VALUES	(201, 'iPhone 14', '501', 1299),
		(202, 'iPhone 8', '502', 999),
		(204, 'iPhone 13', '502', 1199),
		(203, 'iPhone 11', '503', 1199),
		(205, 'iPhone 12', '502', 1199),
		(206, 'iPhone 14', '501', 1399),
		(214, 'iPhone 15', '503', 1499),
		(207, 'iPhone 15', '505', 1499),
		(208, 'iPhone 15', '504', 1499),
		(209, 'iPhone 12', '502', 1299),
		(210, 'iPhone 13', '502', 1199),
		(211, 'iPhone 11', '501', 1099),
		(212, 'iPhone 14', '503', 1399),
		(213, 'iPhone 8', '502', 1099)
;

-- adding more products 

INSERT INTO  productts
VALUES	(222, 'Samsung Galaxy S21', '504', 1699),
		(223, 'Samsung Galaxy S20', '505', 1899),
		(224, 'Google Pixel 6', '501', 899),
		(225, 'Google Pixel 5', '502', 799),
		(226, 'OnePlus 9 Pro', '503', 1699),
		(227, 'OnePlus 9', '502', 1999),
		(228, 'Xiaomi Mi 11', '501', 899),
		(229, 'Xiaomi Mi 10', '504', 699),
		(230, 'Huawei P40 Pro', '505', 1099),
		(231, 'Huawei P30', '502', 1299),
		(232, 'Sony Xperia 1 III', '503', 1199),
		(233, 'Sony Xperia 5 III', '501', 999),
		(234, 'LG Velvet', '505', 1899),
		(235, 'LG G8 ThinQ', '504', 799),
		(236, 'Motorola Edge Plus', '502', 1099),
		(237, 'Motorola One 5G', '501', 799),
		(238, 'ASUS ROG Phone 5', '503', 1999),
		(239, 'ASUS ZenFone 8', '504', 999),
		(240, 'Nokia 8.3 5G', '502', 899),
		(241, 'Nokia 7.2', '501', 699),
		(242, 'BlackBerry Key2', '504', 1899),
		(243, 'BlackBerry Motion', '502', 799),
		(244, 'HTC U12 Plus', '501', 899),
		(245, 'HTC Desire 20 Pro', '505', 699),
		(246, 'Lenovo Legion Phone Duel', '503', 1499),
		(247, 'Lenovo K12 Note', '504', 1499),
		(248, 'ZTE Axon 30 Ultra', '501', 1299),
		(249, 'ZTE Blade 20', '502', 1599),
		(250, 'Oppo Find X3 Pro', '503', 1999);

select * from suppliers;
select * from productts;



-- Write an SQL query to find the name of the product with the highest  price in each country.


select 
		supplier_id,
		supplier_name,
		product_name,
		price,
		country
from
(
	select  
		p.product_name,
		s.supplier_id,
		s.supplier_name,
		s.country,
		p.price,
	-- dense_rank() over(order by p.price desc) as global_rank,
	dense_rank() over(partition by s.country order by p.price desc) as local_rank
	from suppliers as s
 inner join productts as p
on s.supplier_id = p.supplier_id
-- order by 2, 1 desc
) as t1
where local_rank = 1



SELECT 
     supplier_id,
    supplier_name,
    product_id,
    price,
    country
FROM
    (
    SELECT 
        s.supplier_id,
        s.supplier_name,
        s.country,
        p.product_id,
        p.price,
        -- DENSE_RANK() OVER(ORDER BY p.price DESC) as g__rank,
        DENSE_RANK() OVER(PARTITION BY s.country ORDER BY p.price DESC) as d_l_rank
    FROM suppliers as s
    JOIN
    productts as p
    ON s.supplier_id = p.supplier_id
    ) as t2
WHERE d_l_rank = 1

--  day 3  using windows function 



	
DROP TABLE IF EXISTS students;
CREATE TABLE students(id INT,	student_name VARCHAR(15),	sub_1 INT,	sub_2 INT,	sub_3 INT,	sub_4 INT)

insert into students(id,student_name,sub_1,sub_2,sub_3,sub_4)
	values 
	-- (1,'sam',88,85,84,81)
	(2,'rahul',95,91,75,87),
	(3,'john',85,88,87,88),
	(4, 'smith',85,91,86,88)

SELECT *from students;
	
LEAD, LAG

	
SELECT id, student_name, sub_1 as grades FROM students
 -- WHERE sub_1 > sub_2



SELECT 
    id,
	student_name,
	sub_1 as grades,
    LAG(sub_1) OVER(ORDER BY id) as prev_student_grades
FROM students
WHERE grades > prev_student_grades



-- 
select *
	from

(SELECT 
    id,
	student_name,
	sub_1 as grades,
    LAG(sub_1) OVER(ORDER BY id) as prev_student_grades
FROM students) as t3
WHERE grades > prev_student_grades

--  here using the concept of subquery


select *
from
(SELECT 
    id,
	student_name,
	sub_1 as grades,
    Lead(sub_1) OVER(ORDER BY id) as prev_student_grades
FROM students) as t4
WHERE grades > prev_student_grades



--  here the difference of comparing 2 different rows using the concepts of windows function(lag , leap)
--  lead  it will help to find the subsequnet rows queries , i.e future date wthe help of curretnt row data
-- lag measns previuos



--  here using finding the duplicates  concept..... d3N4 file


-- How to find duplicates


INSERT INTO students
VALUES
(3, 'john', 85, 86, 87, 88);

SELECT * FROM 
    (
    SELECT 
        *,
        ROW_NUMBER() OVER(PARTITION BY id, student_name, sub_1, email ORDER BY id) as rn
    FROM students
    ) as t1
WHERE rn = 1

ALTER TABLE students

-- drop column  sub_2
drop column sub_3    
DROP COLUMN sub_4


ALTER TABLE students
ADD COLUMN email VARCHAR(20);


UPDATE students
SET email = 's5@gmail.com'
WHERE  id = 1;

SELECT * FROM students;

INSERT INTO students
VALUES
(1, 'sam', 88, 'rs12@ymail.com');


--  using group by

SELECT 
    id,
    student_name,
    sub_1,
    email,
    COUNT(*)
FROM students
GROUP BY 1, 2, 3, 4
HAVING COUNT(*) > 1



--  CROSS JOIN 
/*
A cross join, also known as a Cartesian join, is a type of join operation in SQL that returns the Cartesian product of two tables. This means it combines each row from the first table with every row from the second table, resulting in a combination of all possible pairs of rows.
*/


DROP TABLE IF EXISTS prooducts;
-- Create the products table
CREATE TABLE prooducts (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50)
);

-- Insert some sample data into the products table
INSERT INTO prooducts (product_id, product_name) VALUES
(1, 'Laptop'),
(2, 'Smartphone'),
(3, 'Tablet');

-- Create the colors table
CREATE TABLE colors (
    color_id INT PRIMARY KEY,
    color_name VARCHAR(20)
);

-- Insert some sample data into the colors table
INSERT INTO colors (color_id, color_name) VALUES
(1, 'Red'),
(2, 'Blue'),
(3, 'Green');


SELECT * FROM prooducts;
SELECT * FROM colors;

-- Retrieve all possible combinations of products and colors using a cross join
SELECT 
    *
FROM prooducts p
CROSS JOIN 
colors c
ORDER BY p.product_name;
    
/*
Expected Output
product_name  | color_name
--------------|-----------
Laptop        | Red
Laptop        | Blue
Laptop        | Green
Smartphone    | Red
Smartphone    | Blue
Smartphone    | Green
Tablet        | Red
Tablet        | Blue
Tablet        | Green
*/

--  INNER JOIN 
DROP TABLE IF EXISTS employees;

CREATE TABLE employees 
    (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(100),
    salary DECIMAL(10, 2),
    manager_id INT
);

INSERT INTO employees (employee_id, employee_name, department, salary, manager_id)
VALUES
    (1, 'John Doe', 'HR', 50000.00, NULL),
    (2, 'Jane Smith', 'HR', 55000.00, 1),
    (3, 'Michael Johnson', 'HR', 60000.00, 1),
    (4, 'Emily Davis', 'IT', 60000.00, NULL),
    (5, 'David Brown', 'IT', 65000.00, 4),
    (6, 'Sarah Wilson', 'Finance', 70000.00, NULL),
    (7, 'Robert Taylor', 'Finance', 75000.00, 6),
    (8, 'Jennifer Martinez', 'Finance', 80000.00, 6);



/*
-- Question
You have a employees table with columns emp_id, emp_name,
department, salary, manager_id (manager is also emp in the table))

Identify employees who have a higher salary than their manager. 
*/


SELECT 
    e1.employee_id,
    e1.employee_name,
    e1.salary,
    e1.manager_id,
    e2.employee_id,
    e2.employee_name as manager_name,
    e2.salary as manager_sale 
FROM employees as e1
LEFT JOIN -- self LEFT JOIN-----  TYPES OF JOI IS  LEFT JOIN
employees as e2
ON e1.manager_id = e2.employee_id
WHERE e1.salary > e2.salary


--  NEED TO CHECK NO MANAGER  THEN?



SELECT 
    e1.employee_id,
    e1.employee_name,
    e1.salary,
    COALESCE(e1.manager_id::TEXT, 'No manager Found'),
    e2.employee_id,
    COALESCE(e2.employee_name, 'No manager') as manager_name,
    e2.salary as manager_sale 
FROM employees as e1
LEFT JOIN -- self 
employees as e2
ON e1.manager_id = e2.employee_id;
--  CAST(e1.manager_id, NUMERIC)
