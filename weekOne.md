**Query #1**
What is the total amount each customer spent at the restaurant?

```    SELECT customer_id, SUM(price)
    FROM dannys_diner.menu a INNER JOIN
    dannys_diner.sales b on a.product_id = b.product_id
    GROUP BY customer_id
    ORDER BY customer_id;
    
```

| customer_id | sum |
| ----------- | --- |
| A           | 76  |
| B           | 74  |
| C           | 36  |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
