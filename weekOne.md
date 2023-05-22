**Query #1**
What is the total amount each customer spent at the restaurant?

```sql 
SELECT customer_id, SUM(price) as Total
FROM dannys_diner.menu a INNER JOIN
dannys_diner.sales b on a.product_id = b.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total |
| ----------- | --- |
| A           | 76  |
| B           | 74  |
| C           | 36  |

---

**Query #2**
How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT (DISTINCT order_date) as days_visited
FROM dannys_diner.sales
GROUP BY customer_id;
```

| customer_id | days_visited |
| ----------- | ------------ |
| A           | 4            |
| B           | 6            |
| C           | 2            |

---

**Query #3**
What was the first item from the menu purchased by each customer?

```sql
WITH my_cte as
  (SELECT *, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) as rowNumber
   FROM dannys_diner.sales a INNER JOIN dannys_diner.menu b
   ON a.product_id = b.product_id)
        
SELECT customer_id, product_name
FROM my_cte
WHERE rowNumber = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | curry        |
| C           | ramen        |

---

**Query #4**
What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT product_name, COUNT(a.product_id) as sale_count
FROM dannys_diner.sales a INNER JOIN dannys_diner.menu b ON a.product_id=b.product_id
GROUP BY product_name
LIMIT 1;
```

| product_name | sale_count |
| ------------ | ---------- |
| ramen        | 8          |

---

**Query #5**
Which item was the most popular for each customer?
```sql
    SELECT customer_id, product_name, amount
    FROM
        (SELECT customer_id, product_name, COUNT(*) as amount, RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(*) DESC) as rank
    	FROM dannys_diner.menu a INNER JOIN dannys_diner.sales b ON a.product_id=b.product_id
    	GROUP BY customer_id, product_name
    	ORDER BY customer_id, amount) as tab
    WHERE rank = 1;
```
| customer_id | product_name | amount |
| ----------- | ------------ | ------ |
| A           | ramen        | 3      |
| B           | ramen        | 2      |
| B           | curry        | 2      |
| B           | sushi        | 2      |
| C           | ramen        | 3      |

---

**Query #6**
Which item was purchased first by the customer after they became a member?
```sql
    SELECT customer_id, join_date, order_date, product_name
    FROM 
    	(SELECT *, ROW_NUMBER() OVER(PARTITION BY join_date ORDER BY join_date, order_date)
    	 FROM
    	     (SELECT a.customer_id, a.join_date, b.order_date, product_name
    	     FROM dannys_diner.members a INNER JOIN dannys_diner.sales b on a.customer_id=b.customer_id
             INNER JOIN dannys_diner.menu c ON b.product_id=c.product_id) as tab
    	     WHERE order_date >= join_date 
    	     ORDER BY join_date, order_date) as tab
     WHERE row_number = 1;
```

| customer_id | join_date                | order_date               | product_name |
| ----------- | ------------------------ | ------------------------ | ------------ |
| A           | 2021-01-07T00:00:00.000Z | 2021-01-07T00:00:00.000Z | curry        |
| B           | 2021-01-09T00:00:00.000Z | 2021-01-11T00:00:00.000Z | sushi        |

---

**Query #7**
Which item was purchased just before the customer became a member?
```sql
     SELECT customer_id, order_date, join_date, product_name
     FROM
         (SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY order_date DESC)
         FROM
	     (SELECT a.customer_id, a.order_date, b.join_date, product_name
	     FROM dannys_diner.sales a INNER JOIN dannys_diner.members b ON a.customer_id=b.customer_id INNER JOIN dannys_diner.menu c ON a.product_id=c.product_id) as                tab
            WHERE order_date < join_date) as tab
     WHERE rank = 1
```

| customer_id | order_date               | join_date                | product_name |
| ----------- | ------------------------ | ------------------------ | ------------ |
| A           | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | sushi        |
| A           | 2021-01-01T00:00:00.000Z | 2021-01-07T00:00:00.000Z | curry        |
| B           | 2021-01-04T00:00:00.000Z | 2021-01-09T00:00:00.000Z | sushi        |

---

**Query #8**

```sql
    SELECT a.customer_id, COUNT(*) as items_count, SUM(price) as total_amount
    FROM dannys_diner.sales a INNER JOIN dannys_diner.menu b ON a.product_id=b.product_id INNER JOIN dannys_diner.members c ON a.customer_id=c.customer_id
    WHERE order_date < join_date
    GROUP BY a.customer_id
    ORDER BY a.customer_id;
```

| customer_id | items_count | total_amount |
| ----------- | ----------- | ------------ |
| A           | 2           | 25           |
| B           | 3           | 40           |
---

**Query #9**
If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
    SELECT customer_id, SUM(total_points) as total_points
    FROM 
    	(SELECT *,
    	CASE WHEN product_name = 'sushi' THEN points * 2
    	ELSE points
    	END as total_points
    	FROM 
    	    (SELECT *,
    	    CASE WHEN price > 0 THEN price * price
            ELSE price
    	    END as points
    	    FROM dannys_diner.sales a INNER JOIN dannys_diner.menu b ON a.product_id=b.product_id) as tab) as tab
    GROUP BY customer_id
    ORDER BY customer_id ASC;
```
| customer_id | total_points |
| ----------- | ---- |
| A           | 1082 |
| B           | 1138 |
| C           | 432  |

---

**Query #10**
In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
    WITH cte as (SELECT *,
    CASE WHEN week_num = 1 THEN points * 2
    ELSE points
    END as total_points
    FROM 
    	(SELECT *,
    	CASE WHEN price > 0 THEN price * price
    	ELSE price
    	END as points
    	FROM
    	    (SELECT a.customer_id, order_date, price, DATE_PART('week', order_date) as week_num
    	    FROM dannys_diner.sales a INNER JOIN dannys_diner.members b 
	    ON a.customer_id=b.customer_id INNER JOIN dannys_diner.menu c ON a.product_id=c.product_id) as tabOne) as tabTwo
    	    WHERE order_date <= '2021-01-31')
    
    SELECT customer_id, sum(total_points) as total_points
    FROM cte
    GROUP BY customer_id;
```
| customer_id | total_points |
| ----------- | ------------ |
| A           | 1351         |
| B           | 894          |

---

[Try Query on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
