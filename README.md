# SQL_Dannys_diner
Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business. (https://8weeksqlchallenge.com/case-study-1/)

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.
Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

## Case Study Questions
1. What is the total amount each customer spent at the restaurant?
````sql
SELECT s.customer_id, SUM(m.price) AS total_spent
FROM Menu m
JOIN sales s ON m.product_id = s.product_id
GROUP BY s.customer_id;
````
| customer_id | total_spent |
|-------------|-------------|
| B           | 74          |
| A           | 76          |
| C           | 36          |

2. How many days has each customer visited the restaurant?
````sql
SELECT s.customer_id AS ID, COUNT(DISTINCT s.order_date) AS No_Visits
FROM sales AS s
INNER JOIN menu AS m ON m.product_id = s.product_id
GROUP BY s.customer_id;
````
| id | no_visits |
|----|-----------|
| C  | 2         |
| A  | 4         |
| B  | 6         |

3. What was the first item from the menu purchased by each customer?
````sql
WITH Rank AS (
  SELECT s.customer_id, 
         m.product_name, 
         s.order_date,
         DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
  FROM sales s
  INNER JOIN menu m ON m.product_id = s.product_id
)
SELECT customer_id, product_name
FROM Rank
WHERE rank = 1;

````
| customer_id | product_name |
|-------------|--------------|
| C           | ramen        |
| A           | curry        |
| A           | sushi        |
| B           | curry        |

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT TOP 1 m.product_name AS Product_Name, COUNT(*) AS No_times_purchased
FROM sales s
INNER JOIN menu m ON m.product_id = s.product_id
GROUP BY m.product_name
ORDER BY COUNT(*) DESC;
````
| product_name | no_times_purchased |
|--------------|--------------------|
| ramen        | 8                  |

5. Which item was the most popular for each customer?
````sql
WITH Ranked AS (
  SELECT s.customer_id, 
         m.product_name, 
         COUNT(*) AS count,
         DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rank
  FROM sales s
  INNER JOIN menu m ON m.product_id = s.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name, count
FROM Ranked
WHERE rank = 1;
````
| customer_id | product_name | count |
|-------------|--------------|-------|
| A           | ramen        | 3     |
| B           | sushi        | 2     |
| B           | ramen        | 2     |
| B           | curry        | 2     |
| C           | ramen        | 3     |

6. Which item was purchased first by the customer after they became a member?
````sql
WITH Ranked AS (
  SELECT s.customer_id, 
         s.order_date, 
         m.product_name,
         DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS rank
  FROM sales s
  INNER JOIN members me ON me.customer_id = s.customer_id
  INNER JOIN menu m ON m.product_id = s.product_id
  WHERE s.order_date >= me.join_date
)
SELECT customer_id, order_date, product_name
FROM Ranked
WHERE rank = 1;
````
| customer_id | order_date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

7. Which item was purchased just before the customer became a member?
````sql
WITH Rank AS (
  SELECT s.customer_id, 
         s.order_date, 
         m.product_name,
         DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank
  FROM sales s
  INNER JOIN members me ON me.customer_id = s.customer_id
  INNER JOIN menu m ON m.product_id = s.product_id
  WHERE s.order_date < me.join_date
)
SELECT customer_id, order_date, product_name
FROM Ranked
WHERE rank = 1;
````
| customer_id | order_date | product_name |
|-------------|------------|--------------|
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |

8. What is the total items and amount spent for each member before they became a member?
````sql
SELECT s.customer_id, 
       COUNT(m.product_name) AS No_Products_Purchased, 
       SUM(m.price) AS Total_spent
FROM sales s
INNER JOIN members me ON me.customer_id = s.customer_id
INNER JOIN menu m ON m.product_id = s.product_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id;
````
| customer_id | no_products_purchased | total_spent |
|-------------|------------------------|-------------|
| B           | 3                      | 40          |
| A           | 2                      | 25          |

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
WITH Points AS (
  SELECT *, 
         CASE 
           WHEN product_id = 1 THEN price * 20
           ELSE price * 10
         END AS points
  FROM menu
)
SELECT s.customer_id, SUM(p.points) AS Points
FROM sales s
JOIN Points p ON p.product_id = s.product_id
GROUP BY s.customer_id;
````
| customer_id | points |
|-------------|--------|
| B           | 940    |
| A           | 860    |
| C           | 360    |

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
SELECT s.customer_id, 
       SUM(CASE 
             WHEN m.product_id = 1 THEN m.price * 20
             WHEN s.order_date BETWEEN me.join_date AND DATEADD(DAY, 6, me.join_date) THEN m.price * 20
             ELSE m.price * 10
           END) AS total_points
FROM sales s
INNER JOIN members me ON me.customer_id = s.customer_id
INNER JOIN menu m ON m.product_id = s.product_id
WHERE s.order_date <= '2021-01-31'
GROUP BY s.customer_id;
````
| customer_id | points |
|-------------|--------|
| A           | 1370   |
| B           | 940    |


## Bonus Questions
Join All The Things
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:
| customer_id | order_date | product_name | price | member |
|-------------|------------|---------------|-------|--------|
| A           | 2021-01-01 | curry         | 15    | N      |
| A           | 2021-01-01 | sushi         | 10    | N      |
| A           | 2021-01-07 | curry         | 15    | Y      |
| A           | 2021-01-10 | ramen         | 12    | Y      |
| A           | 2021-01-11 | ramen         | 12    | Y      |
| A           | 2021-01-11 | ramen         | 12    | Y      |
| B           | 2021-01-01 | curry         | 15    | N      |
| B           | 2021-01-02 | curry         | 15    | N      |
| B           | 2021-01-04 | sushi         | 10    | N      |
| B           | 2021-01-11 | sushi         | 10    | Y      |
| B           | 2021-01-16 | ramen         | 12    | Y      |
| B           | 2021-02-01 | ramen         | 12    | Y      |
| C           | 2021-01-01 | ramen         | 12    | N      |
| C           | 2021-01-01 | ramen         | 12    | N      |
| C           | 2021-01-07 | ramen         | 12    | N      |

````sql
WITH memberstatus AS
(
SELECT s.customer_id,
    CASE
        WHEN me.customer_id IS NOT NULL THEN 'Y'
        ELSE 'N'
    END AS memberstatus
FROM sales AS s
LEFT JOIN MEMBERS AS me ON me.customer_id = s.customer_id
)
SELECT s.customer_id as ID,s.order_date, m.product_name, m.price as Spent, ms.Memberstatus
FROM sales as S
INNER JOIN menu AS m ON m.product_id = s.product_id
INNER JOIN memberstatus AS ms ON ms.customer_id = s.customer_id
GROUP BY s.customer_id, s.order_date, m.product_name, ms.Memberstatus, m.price  
ORDER BY s.customer_id, s.order_date, m.product_name
````
