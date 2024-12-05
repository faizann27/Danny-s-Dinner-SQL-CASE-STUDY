# Danny-s-Dinner-SQL-CASE-STUDY
This case study explores customer purchasing behavior at a restaurant through various SQL queries. The analysis is based on data from three tables: sales, menu, and members. The study covers customer spending habits, frequency of visits, first and most popular items purchased, and their points earned through a loyalty program. 


CREATE DATABASE Dannys_Dinner;
USE Dannys_Dinner;

CREATE TABLE Sales(
Customer_Id VARCHAR(1),
Order_Date DATE,
Product_Id INT
);

INSERT INTO Sales (Customer_Id, Order_Date, Product_Id) VALUES
  ('A', '2021-01-01', 1),
  ('A', '2021-01-01', 2),
  ('A', '2021-01-07', 2),
  ('A', '2021-01-10', 3),
  ('A', '2021-01-11', 3),
  ('A', '2021-01-11', 3),
  ('B', '2021-01-01', 2),
  ('B', '2021-01-02', 2),
  ('B', '2021-01-04', 1),
  ('B', '2021-01-11', 1),
  ('B', '2021-01-16', 3),
  ('B', '2021-02-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-01', 3),
  ('C', '2021-01-07', 3);
  
  CREATE TABLE MENU(
  Product_Id INT,
  Product_Name VARCHAR(50),
  Price int
  );
  
  INSERT INTO MENU
  (Product_Id,Product_Name,Price)
  VALUES
  ('1', 'Sushi', '10'),
  ('2', 'Curry', '15'),
  ('3', 'Ramen', '12');
  
  CREATE TABLE Member(
  Customer_Id VARCHAR(1),
  Join_Date date);
  
  INSERT INTO Member
  (Customer_Id, Join_Date)
  VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
  
  
  CREATE TABLE Member (
    Customer_Id VARCHAR(1),
    Join_Date DATE
);

  
  -- Q1.1. What is the total amount each customer spent at the restaurant?
  
  SELECT
	Customer_Id,
	sum(Price) as Spent
FROM
	Sales s,
	Menu m
WHERE m.Product_Id = s.Product_Id
GROUP BY Customer_Id;

-- Q2. How many days has each customer visited the restaurant?

SELECT
	Customer_Id,
	COUNT(DISTINCT Order_Date) as visits
FROM Sales 
GROUP BY Customer_Id;

-- Q3. What was the first item from the menu purchased by each customer?

Select * from  menu; 
Select * from Sales;

With customer_first_Purchase AS( 
Select s.customer_id, MIN(s.order_date) as First_purchase_date 
from sales s
Group by s.customer_id
)
Select cfp.customer_id,cfp.First_purchase_date, m.product_name
From customer_first_Purchase cfp
inner join sales s on s.customer_id = cfp.customer_id 
AND cfp.First_purchase_date = s.order_date
inner join menu m on m.product_id = s.product_id;


##4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT 
    m.Product_Name,
    COUNT(s.Product_Id) AS Purchase_Count
FROM 
    Sales s
JOIN 
    Menu m ON s.Product_Id = m.Product_Id
GROUP BY 
    m.Product_Name
ORDER BY 
    Purchase_Count DESC
LIMIT 1;

## 5. Which item was the most popular for each customer?


WITH Popular_customer AS (
Select s.customer_id, m.product_name, Count(*) As Purchase_count,
Row_Number() OVER(Partition by s.customer_id order by Count(*) desc) AS Total_Rank
from sales s
Inner join menu m on
s.product_id = m.product_id
group by s.customer_id, m.product_name
)
Select customer_id, product_name, purchase_count
from Popular_customer pc
Where Total_Rank = 1;

##Q.6 Which item was purchased first by the customer after they became a member?##

WITH MEMBERSHIP As
(Select s.customer_id, MIN(s.order_date) As First_Purchase_Date
from sales s
join members ms on
s.customer_id = ms.customer_id
Where s.order_date >= ms.join_date
group by s.customer_id
)
Select msb.customer_id, m.product_name
from MEMBERSHIP msb
join sales s 
on  msb.customer_id = s.customer_id
AND msb.First_Purchase_Date = s.order_date
join menu m on s.product_id = m.product_id;



## 7. Which item was purchased just before the customer became a member?##

WITH BEFORE_MEMBERSHIP AS (
    SELECT 
        s.Customer_Id, 
        MAX(s.Order_Date) AS Last_Purchase_Date
    FROM 
        Sales s
    JOIN 
        Member ms ON s.Customer_Id = ms.Customer_Id
    WHERE 
        s.Order_Date < ms.Join_Date
    GROUP BY 
        s.Customer_Id
)
SELECT 
    bms.Customer_Id, 
    m.Product_Name
FROM 
    BEFORE_MEMBERSHIP bms
JOIN 
    Sales s ON bms.Customer_Id = s.Customer_Id AND bms.Last_Purchase_Date = s.Order_Date
JOIN 
    Menu m ON s.Product_Id = m.Product_Id;
    
    DESCRIBE Sales;
DESCRIBE Member;
DESCRIBE Menu;


##8. What is the total items and amount spent for each member before they became a member?##

SELECT s.customer_id, COUNT(*) as total_items, SUM(m.price) AS total_spent
FROM Sales s
JOIN Menu m ON s.Product_Id = m.Product_Id
JOIN Member mb ON s.Customer_Id = mb.Customer_Id
WHERE s.Order_Date < mb.Join_Date
GROUP BY s.Customer_Id;


##9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier – how many points would each customer have?##

Select s.customer_id,
SUM(CASE when m.product_name = 'sushi' then m.price*20
ELSE m.price*10 END) As TOtal_Point 
from sales s
Join menu m on s.product_id = m.product_id
group by s.customer_id;

## 10. In the first week after a customer joins the program (including their join date) 
## they earn 2x points on all items, not just sushi. How many points do customer A and B have at the end of January?##

SELECT 
    s.customer_id,
    SUM(CASE
        WHEN s.order_date BETWEEN mb.join_date AND DATE_ADD('2021-01-31', interval 7 day) THEN m.price*20
        WHEN m.product_name = 'sushi' THEN m.price*20
        ELSE m.price*10
    END) AS total_points
FROM sales s
        JOIN menu m ON s.product_id = m.product_id
        LEFT JOIN Member mb ON s.customer_id = mb.customer_id
WHERE s.customer_id IN ('A' , 'B') AND s.order_date <= '2021-01-31'
GROUP BY s.customer_id
order by customer_id;
