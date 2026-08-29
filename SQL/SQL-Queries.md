## SQL Queries

use jan_26;
## Q1 a)Fetch the employee number, first name and last name of those employees who are working as Sales Rep reporting to employee with employeenumber 1102 
select * from employees;
select employeenumber,firstname,lastname from employees where jobTitle="sales rep"
and reportsTo=1102;

##b)Show the unique productline values containing the word cars at the end from the products table.
select * from products;
select distinct productline from products where productLine like "%Cars";

## Q2 Using a CASE statement, segment customers into three categories based on their country:(
select * from customers;
select customernumber,country,customername, case when country in ("Usa","Canada")
then "North America" when country in ("UK","France","Germany") then "Europe" else "Others"
end as CountrySegment from customers; 

## Q3 a)Using the OrderDetails table, identify the top 10 products (by productCode) with the highest total order quantity across all orders.
select productcode,sum(quantityordered) total_ordered from orderdetails group by productcode order by total_ordered desc limit 10;


## Q3 b)Company wants to analyse payment frequency by month. Extract the month name from the payment date to count the total number of payments for each month and include only those months with a payment count exceeding 20. Sort the results by total number of payments in descending order.  
select * from payments;
select monthname(paymentDate) payment_month, count(customerNumber) num_payments from payments 
group by monthname(paymentDate) having count(customerNumber)>20 order by num_payments desc;

## Q4 a)Create a table named Customers to store customer information.
create database Customers_Orders;
create table Customers1(customer_id int auto_increment primary key, 
first_name Varchar(50), last_name Varchar(50), email varchar(255), 
phone_number varchar(20));



## b)Create a table named Orders to store information about customer orders. 
create table Orders1( order_id int auto_increment primary key, customer_id int, 
order_date Date, total_amount decimal(10,2),
constraint fk_customer foreign key(customer_id) references customers1 (customer_id),
constraint chk_total_amount check(total_amount>0));

## Q5 List the top 5 countries (by order count) that Classic Models ships to
Select c.country, count(o.orderNumber) order_count from customers c inner join 
orders o on c.customerNumber=o.customerNumber group by 1 order by 2 desc limit 5;


## Q6 Create a table project.Find out the names of employees and their related managers.
Create table project(EmployeeID int primary key auto_increment, Full_name varchar (50),
Gender varchar(10) check(Gender in ('Male', 'Female')), ManagerId int );
insert into project (EmployeeID, Full_name, Gender, ManagerID) values
(1,'Pranaya', 'Male' ,3), (2,'Priyanka', 'Female' ,1), (3,'Preety', 'Female' ,Null), 
(4,'ASnurag', 'Male' ,1),(5,'Sambit', 'Male' ,1),(6,'Rajesh', 'Male' ,3),
(7,'Hina', 'Female' ,3);

select m.Full_Name as 'Manager Name', e.Full_Name as 'Emp Name' from project 
e inner join project m on e.ManagerID=m.EmployeeID;

## Q7 i) Alter the table by adding the primary key and auto increment to Facility_ID column.
ii) Add a new column city after name with data type as varchar which should not accept any null values.
Create table facility ( Facility_id int, Name Varchar(100), State Varchar(100),Country Varchar (100));
Alter Table facility Modify facility_id int primary key auto_increment;
Alter Table facility add city varchar(100)  not null after Name;
desc facility;

## Q8 Create a view named product_category_sales that provides insights into sales performance by product category.
-- CREATE VIEW product_category_sales AS
-- SELECT
-- pl.productLine, sum(od.quantityOrdered * od.priceEach) total_sales,count(distinct o.orderNumber) number_of_orders
-- from productlines pl join products1 p on pl.productLine = p.productLine join orderdetails od
-- on p.productCode = od.productCode join orders o
-- on od.orderNumber = o.orderNumber GROUP BY pl.productLine;

select * from product_category_sales;


## Q9  Create a stored procedure Get_country_payments which takes in year and country as inputs and gives year wise, country wise total amount as an output. Format the total amount to nearest thousand unit (K)
-- CREATE DEFINER=`root`@`localhost` PROCEDURE `Get_country_payments`(in p_year int,in p_country Varchar(50))
-- BEGIN
-- Select Year(p.paymentDate) Year,c.country,CONCAT(ROUND(SUM(p.amount)/1000), 'K') 'Total Amount' FROM customers c
-- Inner join payments p on c.customerNumber = p.customerNumber Where year(p.paymentDate) = p_year and c.country = p_country
-- group by year(p.paymentDate), c.country;
-- END
Call Get_country_payments(2003,'France');

## Q10 a)Rank the customers based on their order frequency
Select c.customerName, count(o.orderNumber)order_count, DENSE_RANK() over (ORDER BY count(o.orderNumber) desc)order_frequency_rank
from customers c join orders o on c.customerNumber = o.customerNumber
GROUP BY c.customerNumber, c.customerName;

## Q10 b)Calculate year wise, month name wise count of orders and year over year (YoY) percentage change. Format the YoY values in no decimals and show in % sign.
With monthly_orders as
(
Select year(orderDate) Year,month(orderDate)Month_No,MONTHNAME(orderDate)Month,
COUNT(orderNumber)as Total_Orders from orders GROUP BY year(orderDate), MONTH(orderDate), MONTHNAME(orderDate))
Select Year,Month,Total_Orders,
ROUND(((Total_Orders -LAG(Total_Orders) OVER (ORDER BY Year, Month_No))* 100.0) /
LAG(Total_Orders) over (ORDER BY Year, Month_No),2)YoY_Percentage_Change
from monthly_orders ORDER BY Year, Month_No;

## Q 11 Find out how many product lines are there for which the buy price value is greater than the average of buy price value. Show the output as product line and its count.
select productLine, count(*) total from products1
where buyPrice >(select avg(buyPrice) from products1) group by productLine;

## Q12 Create a procedure to accept the values for the columns in Emp_EH. Handle the error using exception handling concept. Show the message as “Error occurred” in case of anything wrong.
-- CREATE DEFINER=`root`@`localhost` PROCEDURE `Insert_Emp_EH`(in p_EmpID INT, in p_EmpName VARCHAR(100),
-- in p_EmailAddress VARCHAR(100))
-- BEGIN
-- DECLARE EXIT HANDLER FOR SQLEXCEPTION
-- BEGIN
-- Select 'Error occurred' Message;
-- END;
-- insert into Emp_EH values(p_EmpID, p_EmpName, p_EmailAddress);
-- select 'Record Inserted Successfully' Message;
-- END
Create table emp_EH(EmpID int primary key, EmpName varchar(50), EmailAddress Varchar(100));
Call insert_Emp_EH(1,'samira','samaira@gmail.com');
Call insert_Emp_EH(1,'mira','maira@gmail.com');

## Q13 Create before insert trigger to make sure any new value of Working_hours, if it is negative, then it should be inserted as positive.
-- CREATE TRIGGER trg_working_hours before insert on Emp_BIT for each row
-- BEGIN
-- if NEW.Working_hours < 0 thenSET NEW.Working_hours = Abs (NEW.Working_hours);
-- END if;
-- End
Create table Emp_BIT(Name varchar(50), Occupation varchar(50), working_date Date, working_hours int);
INSERT INTO Emp_BIT VALUES
('Robin', 'Scientist', '2020-10-04', 12),  
('Warner', 'Engineer', '2020-10-04', 10),  
('Peter', 'Actor', '2020-10-04', 13),  
('Marco', 'Doctor', '2020-10-04', 14),  
('Brayden', 'Teacher', '2020-10-04', 12),  
('Antonio', 'Business', '2020-10-04', 11);  

insert into emp_BIT values ('Nancy', 'Nurse','2020-10-04',-5);
select * from emp_BIT;




