# Case Study #1 - Danny's Diner

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

sales
menu
members

**You can inspect the entity relationship diagram and example data below.**
![ERD.png](./ERD.png)

---
### Case Study Questions
Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each \$1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


## I-Connecting to Datbase



```python
%load_ext sql
from sqlalchemy import create_engine

conn_text = 'postgresql://{}:{}@{}/{}'.format(
    pg_user,pg_password,pg_host,pg_db
)
%sql $conn_text
engine = create_engine(conn_text)
```

### Creating DB


```sql
%%sql

DROP SCHEMA IF EXISTS dannys_diner CASCADE;

CREATE SCHEMA IF NOT EXISTS dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE IF NOT EXISTS sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE IF NOT EXISTS menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE IF NOT EXISTS members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

     * postgresql://medhat:***@localhost/medhat
    Done.
    Done.
    Done.
    Done.
    15 rows affected.
    Done.
    3 rows affected.
    Done.
    2 rows affected.





    []



### Verify tables created


```sql
%%sql
SELECT
  	product_id,
    product_name,
    price
FROM dannys_diner.menu
ORDER BY price DESC
LIMIT 5;
```

     * postgresql://medhat:***@localhost/medhat
    3 rows affected.





<table>
    <thead>
        <tr>
            <th>product_id</th>
            <th>product_name</th>
            <th>price</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2</td>
            <td>curry</td>
            <td>15</td>
        </tr>
        <tr>
            <td>3</td>
            <td>ramen</td>
            <td>12</td>
        </tr>
        <tr>
            <td>1</td>
            <td>sushi</td>
            <td>10</td>
        </tr>
    </tbody>
</table>



## II-Answering Questions

1. What is the total amount each customer spent at the restaurant?


```sql
%%sql

SELECT s.customer_id, sum(m.price) AS total_spent
FROM sales AS s
JOIN menu AS m
USING(product_id)
GROUP BY(s.customer_id)
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    3 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>total_spent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>76</td>
        </tr>
        <tr>
            <td>B</td>
            <td>74</td>
        </tr>
        <tr>
            <td>C</td>
            <td>36</td>
        </tr>
    </tbody>
</table>



2. How many days has each customer visited the restaurant?


```sql
%%sql

SELECT s.customer_id, count(DISTINCT order_date) AS number_of_visits
FROM sales AS s
GROUP BY(s.customer_id)
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    3 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>number_of_visits</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>4</td>
        </tr>
        <tr>
            <td>B</td>
            <td>6</td>
        </tr>
        <tr>
            <td>C</td>
            <td>2</td>
        </tr>
    </tbody>
</table>



3. What was the first item from the menu purchased by each customer?


```sql
%%sql

SELECT DISTINCT customer_id, product_name
FROM (
    SELECT s.customer_id, m.product_name,
       DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) AS first_order
    FROM sales AS s
    JOIN menu AS m
    USING(product_id)
) AS t1
WHERE t1.first_order = 1;
```

     * postgresql://medhat:***@localhost/medhat
    4 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>A</td>
            <td>sushi</td>
        </tr>
        <tr>
            <td>B</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>C</td>
            <td>ramen</td>
        </tr>
    </tbody>
</table>



4. What is the most purchased item on the menu and how many times was it purchased by all customers?


```sql
%%sql

SELECT product_name, count(*) AS product_count
FROM sales AS s
JOIN menu AS m
USING(product_id)
GROUP BY 1
LIMIT 1;
```

     * postgresql://medhat:***@localhost/medhat
    1 rows affected.





<table>
    <thead>
        <tr>
            <th>product_name</th>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>ramen</td>
            <td>8</td>
        </tr>
    </tbody>
</table>



5. Which item was the most popular for each customer?


```sql
%%sql

WITH item_rank AS (
    SELECT customer_id,
        product_id,
        count(*), 
        RANK() OVER( PARTITION BY customer_id ORDER BY count(*) DESC )
    FROM sales as s
    GROUP BY customer_id, product_id
)

SELECT customer_id, product_name, count
FROM item_rank as ir
JOIN menu as m
USING(product_id)
WHERE rank = 1
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    5 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>product_name</th>
            <th>count</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
        <tr>
            <td>B</td>
            <td>sushi</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>curry</td>
            <td>2</td>
        </tr>
        <tr>
            <td>B</td>
            <td>ramen</td>
            <td>2</td>
        </tr>
        <tr>
            <td>C</td>
            <td>ramen</td>
            <td>3</td>
        </tr>
    </tbody>
</table>



6. Which item was purchased first by the customer after they became a member?


```sql
%%sql

WITH first_order AS (
    SELECT customer_id, MIN(order_date) AS purchase
    FROM sales as s
    join members as me
    USING(customer_id)
    WHERE order_date >= join_date
    GROUP BY 1
)


SELECT fo.customer_id, purchase, product_name
FROM first_order as fo
JOIN sales as s
ON fo.customer_id=s.customer_id AND fo.purchase=s.order_date
JOIN menu as m
USING(product_id)
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    2 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>purchase</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2021-01-07</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-11</td>
            <td>sushi</td>
        </tr>
    </tbody>
</table>



7. Which item was purchased just before the customer became a member?


```sql
%%sql

WITH first_order AS (
    SELECT customer_id, MAX(order_date) AS purchase
    FROM sales as s
    join members as me
    USING(customer_id)
    WHERE order_date < join_date
    GROUP BY 1
)


SELECT fo.customer_id, purchase, product_name
FROM first_order as fo
JOIN sales as s
ON fo.customer_id=s.customer_id AND fo.purchase=s.order_date
JOIN menu as m
USING(product_id)
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    3 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>purchase</th>
            <th>product_name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2021-01-01</td>
            <td>sushi</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-01</td>
            <td>curry</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-04</td>
            <td>sushi</td>
        </tr>
    </tbody>
</table>



8. What is the total items and amount spent for each member before they became a member?


```sql
%%sql

SELECT customer_id, count(DISTINCT product_id) AS total_items, SUM(price) AS amount_spent
FROM sales as s
JOIN members as me
USING(customer_id)
JOIN menu as m
USING(product_id)
WHERE order_date < join_date
GROUP BY 1
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    2 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>total_items</th>
            <th>amount_spent</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2</td>
            <td>25</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2</td>
            <td>40</td>
        </tr>
    </tbody>
</table>



9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


```sql
%%sql

SELECT customer_id, 
    SUM(
        CASE 
            WHEN product_id = 1 THEN price * 20 
            ELSE price * 10
        END
    ) AS points
FROM sales as s
JOIN menu as m
USING(product_id)
GROUP BY 1
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    3 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>points</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>860</td>
        </tr>
        <tr>
            <td>B</td>
            <td>940</td>
        </tr>
        <tr>
            <td>C</td>
            <td>360</td>
        </tr>
    </tbody>
</table>



10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?


```sql
%%sql

WITH sales_before_end_of_jan AS (
    SELECT * 
    FROM sales 
    
)

SELECT customer_id, 
    SUM(
        CASE 
            WHEN product_id = 1 THEN price * 20
            WHEN order_date BETWEEN join_date AND (join_date + INTERVAL '6 day') THEN price * 20
            ELSE price * 10
        END
    ) AS points
FROM sales as s
JOIN menu as m
USING(product_id)
JOIN members as me
USING(customer_id)
WHERE order_date < '2021-01-01'::timestamp + INTERVAL '1 month'
GROUP BY 1
ORDER BY 1;
```

     * postgresql://medhat:***@localhost/medhat
    2 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>points</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>1370</td>
        </tr>
        <tr>
            <td>B</td>
            <td>820</td>
        </tr>
    </tbody>
</table>



## Bonus Questions

### Join All The Things
The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A | 2021-01-01 | curry | 15 | N |
| A | 2021-01-01 | sushi | 10 | N |
| A | 2021-01-07 | curry | 15 | Y |
| A | 2021-01-10 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| A | 2021-01-11 | ramen | 12 | Y |
| B | 2021-01-01 | curry | 15 | N |
| B | 2021-01-02 | curry | 15 | N |
| B | 2021-01-04 | sushi | 10 | N |
| B | 2021-01-11 | sushi | 10 | Y |
| B | 2021-01-16 | ramen | 12 | Y |
| B | 2021-02-01 | ramen | 12 | Y |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-01 | ramen | 12 | N |
| C | 2021-01-07 | ramen | 12 | N |



```sql
%%sql

SELECT 
    customer_id,
    order_date,
    product_name,
    price,
    CASE
        WHEN join_date IS NOT NULL AND order_date >= join_date THEN 'Y'
        ELSE 'N' END AS member
FROM sales AS s
JOIN menu AS m
USING(product_id)
LEFT JOIN members as me
USING(customer_id)
ORDER BY 1,2;
```

     * postgresql://medhat:***@localhost/medhat
    15 rows affected.





<table>
    <thead>
        <tr>
            <th>customer_id</th>
            <th>order_date</th>
            <th>product_name</th>
            <th>price</th>
            <th>member</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>A</td>
            <td>2021-01-01</td>
            <td>sushi</td>
            <td>10</td>
            <td>N</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-01</td>
            <td>curry</td>
            <td>15</td>
            <td>N</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-07</td>
            <td>curry</td>
            <td>15</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-10</td>
            <td>ramen</td>
            <td>12</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-11</td>
            <td>ramen</td>
            <td>12</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>A</td>
            <td>2021-01-11</td>
            <td>ramen</td>
            <td>12</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-01</td>
            <td>curry</td>
            <td>15</td>
            <td>N</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-02</td>
            <td>curry</td>
            <td>15</td>
            <td>N</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-04</td>
            <td>sushi</td>
            <td>10</td>
            <td>N</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-11</td>
            <td>sushi</td>
            <td>10</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-01-16</td>
            <td>ramen</td>
            <td>12</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>B</td>
            <td>2021-02-01</td>
            <td>ramen</td>
            <td>12</td>
            <td>Y</td>
        </tr>
        <tr>
            <td>C</td>
            <td>2021-01-01</td>
            <td>ramen</td>
            <td>12</td>
            <td>N</td>
        </tr>
        <tr>
            <td>C</td>
            <td>2021-01-01</td>
            <td>ramen</td>
            <td>12</td>
            <td>N</td>
        </tr>
        <tr>
            <td>C</td>
            <td>2021-01-07</td>
            <td>ramen</td>
            <td>12</td>
            <td>N</td>
        </tr>
    </tbody>
</table>




```python

```
