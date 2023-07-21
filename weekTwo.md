
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

**Query #6** What was the maximum number of pizzas delivered in a single order?
```sql
    WITH cte as 
     	(SELECT *, ROW_NUMBER() OVER(PARTITION BY order_id ORDER BY order_id) as order_count
     	FROM pizza_runner.customer_orders)
        
    SELECT order_id, max(order_count) as order_count
    FROM cte
    GROUP BY order_id
    ORDER BY order_count DESC
    LIMIT 1;
```
| order_id | order_count |
| -------- | ----------- |
| 4        | 3           |

---

**Query #8** How many pizzas were delivered that had both exclusions and extras?
```sql
    SELECT COUNT(*) as total_with_exclusions_and_extras
    FROM (SELECT *, 
	 CASE WHEN exclusions = '' THEN 'null' ELSE exclusions END as exclusion,
     CASE WHEN extras = '' THEN 'null' ELSE extras END as extra
     FROM pizza_runner.customer_orders) as tab
    WHERE exclusion != 'null' and extra != 'null'
```
| total_with_exclusions_and_extras |
| -------------------------------- |
| 2                                |

---

**Query #9** What was the total volume of pizzas ordered for each hour of the day?
```sql
    SELECT COUNT(*) as volume, EXTRACT(hour from order_time) as hour_of_day
    FROM pizza_runner.customer_orders
    GROUP BY hour_of_day
    ORDER BY hour_of_day;
```


| volume | hour_of_day |
| ------ | ----------- |
| 1      | 11          |
| 3      | 13          |
| 3      | 18          |
| 1      | 19          |
| 3      | 21          |
| 3      | 23          |

---
**Query #10** What was the volume of orders for each day of the week?
```sql
    SELECT COUNT(*) as volume, EXTRACT('DOW' from order_time) as days
    FROM pizza_runner.customer_orders
    GROUP BY days
    ORDER BY days;
```


| volume | days |
| ------ | ---- |
| 5      | 3    |
| 3      | 4    |
| 1      | 5    |
| 5      | 6    |

---

### B. Runner and Customer Experience
**Query #1** How many runners signed up for each 1 week period?
```sql
    SELECT COUNT(DISTINCT runner_id) as number_of_runners, EXTRACT(week from cast(pickup_time as TIMESTAMP)) as week
    FROM pizza_runner.runner_orders
    WHERE pickup_time != 'null'
    GROUP BY week;
```

| number_of_runners | week |
| ----------------- | ---- |
| 2                 | 1    |
| 3                 | 2    |

---

**Query #2** What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
    WITH cte as (SELECT runner_id, AVG(ABS(EXTRACT(minute FROM order_time) - EXTRACT(minute from pickup))) as avg_time_minutes
    FROM 
    	(SELECT *, CAST(pickup_time as TIMESTAMP) as pickup
    	FROM pizza_runner.runner_orders a LEFT JOIN pizza_runner.customer_orders b ON a.order_id = b.order_id
    	WHERE pickup_time != 'null') as tab
    GROUP BY runner_id
    ORDER BY runner_id)
    
    SELECT runner_id, ROUND(CAST(avg_time_minutes as NUMERIC), 2) as avg_time_minutes
    FROM cte;
```

| runner_id | avg_time_minutes |
| --------- | ----- |
| 1         | 21.67 |
| 2         | 27.80 |
| 3         | 10.00 |

---

**Query #4** What was the average distance traveled for each customer?
```sql
    SELECT customer_id, ROUND(AVG(CAST(SUBSTRING(distance, '[0-9.]+') as NUMERIC)),2) as avg_distance_km
    FROM pizza_runner.runner_orders a LEFT JOIN pizza_runner.customer_orders b ON a.order_id=b.order_id
    GROUP by customer_id
    ORDER BY customer_id;
```

| customer_id | avg_distance_km |
| ----------- | --------------- |
| 101         | 20.00           |
| 102         | 16.73           |
| 103         | 23.40           |
| 104         | 10.00           |
| 105         | 25.00           |

---

**Query #5** What was the difference between the longest and shortest delivery times for all orders?

```sql
    SELECT MAX(durations) - MIN(durations) as difference
    FROM(
      SELECT *, CAST(SUBSTRING(duration, '[0-9]+') AS NUMERIC) as durations
    	FROM pizza_runner.runner_orders) as tab;
```

| difference |
| ---------- |
| 30         |

---

**Query #6** What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
    SELECT runner_id, order_id, ROUND(AVG(distances / times),2) as speed
    FROM
    	(SELECT *, CAST(SUBSTRING(distance, '[0-9.]+') AS NUMERIC)*1000 as distances, CAST(SUBSTRING(duration, '[0-9.]+') AS NUMERIC)*60 as times
    	FROM pizza_runner.runner_orders
    	WHERE distance != 'null' and duration != 'null') as tab
    GROUP BY runner_id, order_id
    ORDER BY runner_id, order_id;
```

| runner_id | order_id | speed |
| --------- | -------- | ----- |
| 1         | 1        | 10.42 |
| 1         | 2        | 12.35 |
| 1         | 3        | 11.17 |
| 1         | 10       | 16.67 |
| 2         | 4        | 9.75  |
| 2         | 7        | 16.67 |
| 2         | 8        | 26.00 |
| 3         | 5        | 11.11 |

---

**Query #7** What is the successful delivery percentage for each runner?

```sql
    SELECT runner_id, SUM(completed) * 100 /COUNT(runner_id) as percentage_completed
    FROM 
    	(SELECT *,
    	CASE WHEN cancellation = 'null' or cancellation = '' or cancellation IS NULL THEN 1 ELSE 0 END as completed
    	FROM pizza_runner.runner_orders) as tab
    GROUP BY runner_id
    ORDER BY runner_id;
```

| runner_id | percentage_completed |
| --------- | -------------------- |
| 1         | 100                  |
| 2         | 75                   |
| 3         | 50                   |

---

### C. Ingredient Optimisation
**Query #1** What are the standard ingredients for each pizza?

```sql
SELECT pn.pizza_name AS pizza_name, pt.topping_name AS standard_ingredient
FROM pizza_names pn
JOIN pizza_recipes pr ON pn.pizza_id = pr.pizza_id JOIN pizza_toppings pt ON pt.topping_id = ANY (string_to_array(pr.toppings, ', '))
ORDER BY pn.pizza_id, pt.topping_id;
```

**Query #2** What was the most commonly added extra?

```sql
SELECT extras AS added_extra, COUNT(*) AS count
FROM customer_orders
WHERE extras IS NOT NULL
GROUP BY extras
ORDER BY COUNT(*) DESC
LIMIT 1;
```

**Query #3** What was the most common exclusion?
```sql
SELECT exclusions, COUNT(*) AS count
FROM customer_orders
WHERE exclusions IS NOT NULL
GROUP BY exclusions
ORDER BY COUNT(*) DESC
LIMIT 1;
```

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
