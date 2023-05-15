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
GROUP BY customer_id
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

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
