
### A. Pizza Metrics
**Query #1** How many pizzas were ordered?
```sql
    SELECT COUNT(*) as total_number_of_orders
    FROM pizza_runner.customer_orders;
```
| total_number_of_orders |
| ---------------------- |
| 14                     |

---

**Query #2** How many unique customer orders were made?
```sql
    SELECT COUNT (DISTINCT order_id) as unique_orders
    FROM pizza_runner.customer_orders;
```
| unique_orders |
| ------------- |
| 10            |

---

**Query #3** How many successful orders were delivered by each runner?
```sql
    SELECT runner_id, COUNT(*) as completed_orders
    FROM pizza_runner.runner_orders
    WHERE pickup_time != 'null'
    GROUP BY runner_id;
```
| runner_id | completed_orders |
| --------- | ---------------- |
| 3         | 1                |
| 2         | 3                |
| 1         | 4                |

---

**Query #4** How many of each type of pizza was delivered?
```sql
    SELECT pizza_name, COUNT(a.customer_id) as count
    FROM customer_orders a INNER JOIN pizza_names b ON a.pizza_id=b.pizza_id
    GROUP BY pizza_name;
```

| pizza_name | count |
| --------- | ---------------- |
| Meatlovers         | 10                |
| Vegetarian         | 4               |

---

**Query #5** How many Vegetarian and Meatlovers were ordered by each customer?
```sql
    SELECT a.customer_id, pizza_name, COUNT(a.customer_id)
    FROM pizza_runner.customer_orders a INNER JOIN pizza_runner.pizza_names b ON a.pizza_id=b.pizza_id
    GROUP BY a.customer_id, pizza_name
    ORDER BY a.customer_id;
```
| customer_id | pizza_name | count |
| ----------- | ---------- | ----- |
| 101         | Meatlovers | 2     |
| 101         | Vegetarian | 1     |
| 102         | Meatlovers | 2     |
| 102         | Vegetarian | 1     |
| 103         | Meatlovers | 3     |
| 103         | Vegetarian | 1     |
| 104         | Meatlovers | 3     |
| 105         | Vegetarian | 1     |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
