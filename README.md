🍜 Case Study #1: Danny's Diner
                                                                                      
## Case Study Questions
Each of the following case study questions can be answered using a single SQL statement:

1 What is the total amount each customer spent at the restaurant?

**Query #1**

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

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)


2 How many days has each customer visited the restaurant?

3 What was the first item from the menu purchased by each customer?

4 What is the most purchased item on the menu and how many times was it purchased by all customers?

5 Which item was the most popular for each customer?

6 Which item was purchased first by the customer after they became a member?

7 Which item was purchased just before the customer became a member?

8 What is the total items and amount spent for each member before they became a member?

9 If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

10 In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
