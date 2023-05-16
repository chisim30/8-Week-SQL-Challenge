**Query #1**
What is the total amount each customer spent at the restaurant?

```  
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

```
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

```
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
```
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
```
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

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
