# 🍜 Danny’s Diner SQL Case Study

This project is part of the [8 Week SQL Challenge](https://8weeksqlchallenge.com/case-study-1/).  
It explores customer behavior, spending habits, and menu popularity at **Danny’s Diner** using SQL.

---

## 📂 Dataset
The case study uses three tables:

- **sales** – records of customer purchases  
- **menu** – product details and prices  
- **members** – customer membership join dates  

---

## 📝 Case Study Questions & Solutions

### 1. Total Amount Spent by Each Customer
```sql
Select s.customer_id,
        SUM(m.price)
 FROM dannys_diner.sales s
 Join dannys_diner.menu m
 ON m.product_id = s.product_id
 Group by 1
 Order by customer_id ASC;
```

### 2. Number of Days Visited By Each Customer
```sql
Select customer_id,
	Count(DISTINCT(order_date))
From dannys_diner.sales
group by customer_id;
```

### 3. First Item Purchased by Each Customer
```sql
with CTE as (
Select s.customer_id,
     s.order_date,
     m.product_name,
     Row_Number() OVER (Partition by s.customer_id order by s.order_date) as rank
FROM dannys_diner.sales s
Join dannys_diner.menu m
ON m.product_id = s.product_id
)

select customer_id,
	product_name
From CTE
Where rank = '1';
```

### 4. Most Purchased Item Overall & How Many Times Was It Purchased by Each Customer
```sql
Select COUNT(s.product_id) AS number_sold,
	m.product_name
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
group by m.product_name
ORDER BY number_sold desc
LIMIT 1;
```

### 5. Most Popular Item on the Menu for Each Customer
```sql
WITH CTE AS (
Select s.customer_id,
	COUNT(s.product_id) AS number_sold,
	m.product_name,
    Row_Number() OVER (Partition by s.customer_id order by COUNT(s.product_id) desc) as rank
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
group by s.customer_id, m.product_name
order by customer_id asc, number_sold desc
)

Select customer_id,
	product_name
FROM CTE
WHERE rank = 1;
```

### 6. Item First Purchased by Each Customer After Becoming A Member
```sql
WITH CTE AS (
SELECT s.customer_id,
	s.order_date,
    s.product_id,
    m.product_name,
    j.join_date,
    Row_Number() OVER (Partition by s.customer_id order by s.order_date asc) as rank
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
JOIN dannys_diner.members j
ON s.customer_id = j.customer_id
WHERE order_date >= join_date
)

 SELECT customer_id,
  	product_name
  FROM CTE
  WHERE rank = 1;
```

### 7. Last Item Purchased Before Becoming a Member
```sql
WITH CTE AS (
SELECT s.customer_id,
	s.order_date,
    s.product_id,
    m.product_name,
    j.join_date,
    Row_Number() OVER (Partition by s.customer_id order by s.order_date desc) as rank
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
JOIN dannys_diner.members j
ON s.customer_id = j.customer_id
WHERE order_date < join_date
)

SELECT customer_id,
	product_name
FROM CTE
WHERE rank = 1;
```

### 8. Total Items and Amount Spent Before Membership
```sql
SELECT s.customer_id,
    COUNT(m.product_name) AS total_items,
    SUM(m.price) AS total_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
JOIN dannys_diner.members j
ON s.customer_id = j.customer_id
WHERE s.order_date < j.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id asc;
```

### 9. Points Earned by Each Customer
##### Each $1 Spent Equates to 10 Points and Sushi has a 2x Points Multiplier
```sql
SELECT s.customer_id,
	SUM(CASE
    WHEN m.product_id = 1 THEN m.price * 20
    ELSE m.price * 10
    END) AS points
FROM dannys_diner.menu m
JOIN dannys_diner.sales s
ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```

### 10. Points Earned by Customers A and B by the end of January?
##### In the first week after joining the program (including their join date) they earn 2x points on all items, not just sushi
```sql
 SELECT s.customer_id,
	SUM(CASE
    WHEN s.order_date BETWEEN j.join_date AND (j.join_date + 6) THEN m.price * 20
    WHEN s.product_id = 1 THEN m.price * 20
    ELSE m.price *10
    END) AS points
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
ON m.product_id = s.product_id
JOIN dannys_diner.members j
ON s.customer_id = j.customer_id
WHERE DATE_TRUNC('month',s.order_date) = '2021-01-01'
GROUP BY s.customer_id
ORDER BY s.customer_id ASC;
```
