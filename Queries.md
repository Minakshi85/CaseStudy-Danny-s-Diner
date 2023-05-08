🍜 Case Study #1: Danny's Diner

## Schema (PostgreSQL v13)**

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
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
     
    
    CREATE TABLE menu (
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
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---
                                                                                     
## Case Study Questions
Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?

**Query #1** With Group by and Order by Clause

    SELECT
      	customer_id, sum(price) as total_amount
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    GROUP BY customer_id
    ORDER BY customer_id ;

| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---


2. How many days has each customer visited the restaurant?

**Query #2** With Group by and Order by Clause

    SELECT
      	customer_id, COUNT(DISTINCT order_date) as Visits
    FROM dannys_diner.sales s
    GROUP BY customer_id
    ORDER BY customer_id ;

| customer_id | visits |
| ----------- | ------ |
| A           | 4      |
| B           | 6      |
| C           | 2      |

---


3. What was the first item from the menu purchased by each customer?

**Query #3.1** USING Sub-query

  SELECT
        customer_id, product_name
  FROM dannys_diner.sales s
  LEFT JOIN dannys_diner.menu m
  ON m.product_id = s.product_id
  WHERE (customer_id, order_date) IN (SELECT customer_id, MIN(order_date)
        									FROM dannys_diner.sales
        									GROUP BY customer_id)
  GROUP BY customer_id, product_name
  ORDER BY customer_id;

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---

**Query #3.2** With DENSE_RANK() Window Function

    WITH CTE AS(
    SELECT
      	customer_id, product_name,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS drnk
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id)
    
    Select customer_id, product_name 
    FROM CTE
    WHERE drnk = 1
    GROUP BY  customer_id, product_name;

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---


4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Query #4**

    SELECT
      	product_name, count(s.product_id) as most_purchased
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    GROUP BY product_name
    LIMIT 1;

| product_name | most_purchased |
| ------------ | -------------- |
| ramen        | 8              |

---

5. Which item was the most popular for each customer?

**Query #5**

    WITH PopularItem as
    (SELECT
      	customer_id, product_name, count(s.product_id) as purchase_count,
    	DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY count(s.product_id) DESC) as rnk
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    GROUP BY customer_id, product_name)
    
    SELECT customer_id, product_name
    FROM PopularItem
    WHERE rnk = 1;

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |

---

6. Which item was purchased first by the customer after they became a member?

**Query #6**

    WITH firstpurchase as
      (SELECT
          s.customer_id, product_name, s.order_date,
          DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) as rnk
      FROM dannys_diner.sales s
      LEFT JOIN dannys_diner.menu m
      ON m.product_id = s.product_id
      LEFT JOIN dannys_diner.members mem
      ON mem.customer_id = s.customer_id
      WHERE order_date >= join_date)
    SELECT
      	customer_id, product_name, order_date FROM firstpurchase
    WHERE rnk =1;

| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | curry        | 2021-01-07T00:00:00.000Z |
| B           | sushi        | 2021-01-11T00:00:00.000Z |

---

7. Which item was purchased just before the customer became a member?

**Query #7**

    WITH firstpurchase as
      (SELECT
          s.customer_id, product_name, s.order_date,
          DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date) as rnk
      FROM dannys_diner.sales s
      LEFT JOIN dannys_diner.menu m
      ON m.product_id = s.product_id
      LEFT JOIN dannys_diner.members mem
      ON mem.customer_id = s.customer_id
      WHERE order_date <= join_date)
    SELECT
      	customer_id, product_name, order_date FROM firstpurchase
    WHERE rnk =1;

| customer_id | product_name | order_date               |
| ----------- | ------------ | ------------------------ |
| A           | sushi        | 2021-01-01T00:00:00.000Z |
| A           | curry        | 2021-01-01T00:00:00.000Z |
| B           | curry        | 2021-01-01T00:00:00.000Z |

---

8. What is the total items and amount spent for each member before they became a member?

**Query #8**

    SELECT
        s.customer_id, COUNT(product_name) AS product_count, SUM(m.price) AS total_price
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    LEFT JOIN dannys_diner.members mem
    ON mem.customer_id = s.customer_id
    WHERE order_date < join_date
    GROUP BY s.customer_id;

| customer_id | product_count | total_price |
| ----------- | ------------- | ----------- |
| A           | 2             | 25          |
| B           | 3             | 40          |

---

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

**Query #9**

    SELECT
        s.customer_id,  
    SUM(CASE WHEN m.product_name ='sushi' THEN 20*m.price ELSE 10*m.price END) as points
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    LEFT JOIN dannys_diner.members mem
    ON mem.customer_id = s.customer_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
