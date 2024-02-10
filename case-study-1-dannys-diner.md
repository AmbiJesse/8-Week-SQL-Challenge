# Case Study #1: Danny's Diner
<img width='500' src= "https://8weeksqlchallenge.com/images/case-study-designs/1.png">

## Table of Contents
- [Problem Statement](https://github.com/AmbiJesse/8-Week-SQL-Challenge/edit/main/case-study-1-dannys-diner.md#problem-statement)
- [Entity Relationship Diagram](https://github.com/AmbiJesse/8-Week-SQL-Challenge/edit/main/case-study-1-dannys-diner.md#entity-relationship-diagram-erd)
- [Solutions](https://github.com/AmbiJesse/8-Week-SQL-Challenge/edit/main/case-study-1-dannys-diner.md#solutions)

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favorite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers. He plans on using these insights to help him decide whether he should expand the existing customer loyalty program.

### Entity Relationship Diagram (ERD)
<img width="702" alt="Screenshot 2024-02-09 at 10 01 30 PM" src="https://github.com/AmbiJesse/8-Week-SQL-Challenge/assets/21045393/c4345af6-8893-4785-ba96-e7d9faa9587a">

## Solutions
There are 10 questions to be answers and solved using SQL.

**1. What is the total amount each customer spent at the restaurant?**
```SQL
select customer_id, sum(price) as total_spent
from dannys_diner.sales as s
inner join dannys_diner.menu as m
  using(product_id)
group by customer_id
order by total_spent desc;
```
Result:
| customer_id | total_spent |
| ----------- | ----------- |
| A | 76 |
| B | 74 |
| C | 36 |

- Customer A spent the most at $76, Customer B spent $74 and Customer C spent $36.
---
2. How many days has each customer visited the restaurant?
```SQL
select customer_id, count(distinct order_date) as visit_count
from dannys_diner.sales
group by customer_id
order by visit_count desc;
```
Result:
| customer_id | visit_count |
| ----------- | ----------- |
| B | 6 |
| A | 4 |
| C | 2 |

- Utilized `distinct` to avoid duplicates of multiple orders made on the same day.
- Ordered results in descending order to see customers who visit the most, first.
---
3. What was the first item from the menu purchased by each customer?
```SQL
select
```
---
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```SQL
select
```
---
5. Which item was the most popular for each customer?
```SQL
select
```
---
6. Which item was purchased first by the customer after they became a member?
```SQL
select
```
---
7. Which item was purchased just before the customer became a member?
```SQL
select
```
---
8. What is the total items and amount spent for each member before they became a member?
```SQL
select
```
---
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```SQL
select
```
---
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```SQL
select
```
---
