# SQL Queries

USE jan_26;

## Q1 (a)

**Fetch the employee number, first name and last name of those employees who are working as Sales Rep reporting to employee with EmployeeNumber 1102.**

```sql
SELECT * FROM employees;

SELECT employeenumber, firstname, lastname
FROM employees
WHERE jobTitle = "sales rep"
AND reportsTo = 1102;
```

## Q1 (b)

**Show the unique productline values containing the word cars at the end from the products table.**

```sql
SELECT * FROM products;

SELECT DISTINCT productline
FROM products
WHERE productLine LIKE "%Cars";
```

## Q2

**Using a CASE statement, segment customers into three categories based on their country.**

```sql
SELECT * FROM customers;

SELECT 
    customernumber,
    country,
    customername,
    CASE 
        WHEN country IN ("Usa", "Canada") THEN "North America"
        WHEN country IN ("UK", "France", "Germany") THEN "Europe"
        ELSE "Others"
    END AS CountrySegment
FROM customers;
```

## Q3 (a)

**Using the OrderDetails table, identify the top 10 products (by productCode) with the highest total order quantity across all orders.**

```sql
SELECT 
    productcode,
    SUM(quantityordered) AS total_ordered
FROM orderdetails
GROUP BY productcode
ORDER BY total_ordered DESC
LIMIT 10;
```

## Q3 (b)

**Company wants to analyse payment frequency by month. Extract the month name from the payment date to count the total number of payments for each month and include only those months with a payment count exceeding 20. Sort the results by total number of payments in descending order.**

```sql
SELECT * FROM payments;

SELECT 
    MONTHNAME(paymentDate) AS payment_month,
    COUNT(customerNumber) AS num_payments
FROM payments
GROUP BY MONTHNAME(paymentDate)
HAVING COUNT(customerNumber) > 20
ORDER BY num_payments DESC;
```

## Q4 (a)

**Create a table named Customers to store customer information.**

```sql
CREATE DATABASE Customers_Orders;

CREATE TABLE Customers1 (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(255),
    phone_number VARCHAR(20)
);
```

## Q4 (b)

**Create a table named Orders to store information about customer orders.**

```sql
CREATE TABLE Orders1 (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    CONSTRAINT fk_customer 
        FOREIGN KEY (customer_id) 
        REFERENCES customers1(customer_id),
    CONSTRAINT chk_total_amount 
        CHECK(total_amount > 0)
);
```

## Q5

**List the top 5 countries (by order count) that Classic Models ships to.**

```sql
SELECT 
    c.country,
    COUNT(o.orderNumber) AS order_count
FROM customers c
INNER JOIN orders o 
    ON c.customerNumber = o.customerNumber
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```

## Q6

**Create a table project. Find out the names of employees and their related managers.**

```sql
CREATE TABLE project (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    Full_name VARCHAR(50),
    Gender VARCHAR(10) CHECK(Gender IN ('Male', 'Female')),
    ManagerId INT
);

INSERT INTO project 
(EmployeeID, Full_name, Gender, ManagerID) 
VALUES
(1, 'Pranaya', 'Male', 3),
(2, 'Priyanka', 'Female', 1),
(3, 'Preety', 'Female', NULL),
(4, 'ASnurag', 'Male', 1),
(5, 'Sambit', 'Male', 1),
(6, 'Rajesh', 'Male', 3),
(7, 'Hina', 'Female', 3);

SELECT 
    m.Full_Name AS 'Manager Name',
    e.Full_Name AS 'Emp Name'
FROM project e
INNER JOIN project m 
    ON e.ManagerID = m.EmployeeID;
```

## Q7

**(i) Alter the table by adding the primary key and auto increment to Facility_ID column.**

**(ii) Add a new column city after name with data type as varchar which should not accept any null values.**

```sql
CREATE TABLE facility (
    Facility_id INT,
    Name VARCHAR(100),
    State VARCHAR(100),
    Country VARCHAR(100)
);

ALTER TABLE facility 
MODIFY facility_id INT PRIMARY KEY AUTO_INCREMENT;

ALTER TABLE facility 
ADD city VARCHAR(100) NOT NULL AFTER Name;

DESC facility;
```

## Q8

**Create a view named product_category_sales that provides insights into sales performance by product category.**

```sql
-- CREATE VIEW product_category_sales AS
-- SELECT
--     pl.productLine,
--     SUM(od.quantityOrdered * od.priceEach) total_sales,
--     COUNT(DISTINCT o.orderNumber) number_of_orders
-- FROM productlines pl 
-- JOIN products1 p 
--     ON pl.productLine = p.productLine 
-- JOIN orderdetails od 
--     ON p.productCode = od.productCode 
-- JOIN orders o 
--     ON od.orderNumber = o.orderNumber 
-- GROUP BY pl.productLine;

SELECT * FROM product_category_sales;
```

## Q9

**Create a stored procedure Get_country_payments which takes in year and country as inputs and gives year wise, country wise total amount as an output. Format the total amount to nearest thousand unit (K).**

```sql
-- CREATE DEFINER=`root`@`localhost` PROCEDURE `Get_country_payments`
-- (in p_year int, in p_country Varchar(50))
-- BEGIN
-- Select Year(p.paymentDate) Year,
-- c.country,
-- CONCAT(ROUND(SUM(p.amount)/1000), 'K') 'Total Amount'
-- FROM customers c
-- Inner join payments p 
--     on c.customerNumber = p.customerNumber 
-- Where year(p.paymentDate) = p_year 
--     and c.country = p_country
-- group by year(p.paymentDate), c.country;
-- END

CALL Get_country_payments(2003, 'France');
```

## Q10 (a)

**Rank the customers based on their order frequency.**

```sql
SELECT 
    c.customerName,
    COUNT(o.orderNumber) AS order_count,
    DENSE_RANK() OVER (
        ORDER BY COUNT(o.orderNumber) DESC
    ) AS order_frequency_rank
FROM customers c 
JOIN orders o 
    ON c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName;
```

## Q10 (b)

**Calculate year wise, month name wise count of orders and year over year (YoY) percentage change. Format the YoY values in no decimals and show in % sign.**

```sql
WITH monthly_orders AS
(
    SELECT 
        YEAR(orderDate) AS Year,
        MONTH(orderDate) AS Month_No,
        MONTHNAME(orderDate) AS Month,
        COUNT(orderNumber) AS Total_Orders 
    FROM orders 
    GROUP BY 
        YEAR(orderDate), 
        MONTH(orderDate), 
        MONTHNAME(orderDate)
)
SELECT 
    Year,
    Month,
    Total_Orders,
    ROUND(
        (
            (Total_Orders - LAG(Total_Orders) OVER 
                (ORDER BY Year, Month_No)) * 100.0
        ) /
        LAG(Total_Orders) OVER 
            (ORDER BY Year, Month_No),
        2
    ) AS YoY_Percentage_Change
FROM monthly_orders 
ORDER BY Year, Month_No;
```

## Q11

**Find out how many product lines are there for which the buy price value is greater than the average of buy price value. Show the output as product line and its count.**

```sql
SELECT 
    productLine,
    COUNT(*) AS total
FROM products1
WHERE buyPrice > (
    SELECT AVG(buyPrice) 
    FROM products1
)
GROUP BY productLine;
```

## Q12

**Create a procedure to accept the values for the columns in Emp_EH. Handle the error using exception handling concept. Show the message as “Error occurred” in case of anything wrong.**

```sql
-- CREATE DEFINER=`root`@`localhost` PROCEDURE `Insert_Emp_EH`
-- (in p_EmpID INT, in p_EmpName VARCHAR(100),
--  in p_EmailAddress VARCHAR(100))
-- BEGIN
-- DECLARE EXIT HANDLER FOR SQLEXCEPTION
-- BEGIN
-- Select 'Error occurred' Message;
-- END;
-- insert into Emp_EH 
-- values(p_EmpID, p_EmpName, p_EmailAddress);
-- select 'Record Inserted Successfully' Message;
-- END

CREATE TABLE emp_EH (
    EmpID INT PRIMARY KEY,
    EmpName VARCHAR(50),
    EmailAddress VARCHAR(100)
);

CALL insert_Emp_EH(
    1,
    'samira',
    'samaira@gmail.com'
);

CALL insert_Emp_EH(
    1,
    'mira',
    'maira@gmail.com'
);
```

## Q13

**Create before insert trigger to make sure any new value of Working_hours, if it is negative, then it should be inserted as positive.**

```sql
-- CREATE TRIGGER trg_working_hours 
-- before insert on Emp_BIT 
-- for each row
-- BEGIN
-- if NEW.Working_hours < 0 then
-- SET NEW.Working_hours = Abs(NEW.Working_hours);
-- END if;
-- End

CREATE TABLE Emp_BIT (
    Name VARCHAR(50),
    Occupation VARCHAR(50),
    working_date DATE,
    working_hours INT
);

INSERT INTO Emp_BIT VALUES
('Robin', 'Scientist', '2020-10-04', 12),
('Warner', 'Engineer', '2020-10-04', 10),
('Peter', 'Actor', '2020-10-04', 13),
('Marco', 'Doctor', '2020-10-04', 14),
('Brayden', 'Teacher', '2020-10-04', 12),
('Antonio', 'Business', '2020-10-04', 11);

INSERT INTO emp_BIT VALUES 
('Nancy', 'Nurse', '2020-10-04', -5);

SELECT * FROM emp_BIT;
```
