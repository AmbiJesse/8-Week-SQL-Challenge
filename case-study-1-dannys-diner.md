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
There are 10 questions to be answered and solved using SQL.

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
**2. How many days has each customer visited the restaurant?**
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
**3. What was the first item from the menu purchased by each customer?**
```SQL
with order_sequence as (
    select
        customer_id,
        order_date,
        product_name,
        dense_rank() over(partition by customer_id
                          order by order_date) as rank
    from dannys_diner.sales
    inner join dannys_diner.menu using(product_id)
)
select
    customer_id,
    product_name as first_item_ordered
from order_sequence
where rank = 1
group by customer_id, product_name
order by customer_id;
```
Result:
| customer_id | first_item_ordered |
| ----------- | ----------- |
| A | curry |
| A | sushi |
| B | curry |
| C | ramen |

- Used a Common Table Expression (CTE) named `order_sequence` to `rank` the order_dates sequentially to filter for the first orders made.
- With data limitations of context for orders made on the same day, `dense_rank` is used to categorize both items purchased for Customer A as their first item(s) ordered. The data states that Customer A ordered both curry and sushi on `2021-01-01`.
---
**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```SQL
select
    product_name,
    count(s.product_id) as order_count
from dannys_diner.sales as s
inner join dannys_diner.menu as m using(product_id)
group by product_name
order by order_count desc
limit 1;
```
Result:
| product_name | order_count |
| ----------- | ----------- |
| ramen | 8 |

- Used `count` to aggregate the number of `product_id`s present in every sale, grouped by the menu item name, and ordered by the `order_count` to find that ramen was ordered by all the customers eight times.
---
**5. Which item was the most popular for each customer?**
```SQL
with ranked_orders as (
    select
        customer_id,
        product_name,
        count(s.product_id) as order_count,
        dense_rank() over (partition by customer_id
                           order by count(s.product_id) desc) as rank
    from dannys_diner.sales as s
    inner join dannys_diner.menu as m using(product_id)
    group by customer_id, product_name
)
select
    customer_id,
    product_name,
    order_count
from ranked_orders
where rank = 1;
```
Result:
| customer_id | product_name | order_count |
| ----------- | ----------- | -- |
| A | ramen | 3 |
| B | sushi | 2 |
| B | curry | 2 |
| B | ramen | 2 |
| C | ramen | 3 |

- Used the same structure as question #3 with a Common Table Expression to rank the most ordered items with `dense_rank` which displays multiple items for Customer B. Other methods are available such as `row_number` but that would simply take the first item as being Customer B's favorite item to order.
---
**6. Which item was purchased first by the customer after they became a member?**
```SQL
with memb_orders as (
    select
        s.customer_id,
        join_date,
        order_date,
        s.product_id,
        product_name,
        row_number() over (partition by s.customer_id
                           order by order_date) as rank
    from dannys_diner.sales as s
    inner join dannys_diner.members as mb using(customer_id)
    inner join dannys_diner.menu as mn using(product_id)
    where order_date >= join_date
)
select
    customer_id as customer,
    join_date as joined,
    order_date as first_order,
    product_name as menu_item
from memb_orders
where rank = 1;
```
Result:
| customer | joined | first_order | menu_item |
| ----------- | ----------- | -- | -- |
| A | 2021-01-07 | 2021-01-07 | curry |
| B | 2021-01-09 | 2021-01-11 | sushi |

- The discrepancy to address here is that there is no distinction between `join_date` time to understand if order was placed before or after the customer joined that day. As logged as `join_date` and `order_date`, the items should be understood simultaneously which places the order of Customer A as 'curry' and not using a hard 'after' they joined.

---
**7. Which item was purchased just before the customer became a member?**
```SQL
with pre_memb_orders as (
    select
        s.customer_id,
        join_date,
        order_date,
        s.product_id,
        product_name,
        row_number() over (partition by s.customer_id
                           order by order_date desc) as rank
    from dannys_diner.sales as s
    inner join dannys_diner.members as mb using(customer_id)
    inner join dannys_diner.menu as mn using(product_id)
    where order_date < join_date
)
select
    customer_id as customer,
    order_date as last_order,
    join_date as joined,
    product_name as menu_item
from pre_memb_orders
where rank = 1;
```
Result:
| customer | last_order | joined | menu_item |
| ----------- | ----------- | -- | -- |
| A | 2021-01-01 | 2021-01-07 | sushi |
| A | 2021-01-01 | 2021-01-07 | curry |
| B | 2021-01-04 | 2021-01-09 | sushi |

- Similar to question #7's solution, the difference made is to filter for `order_date`s before the `join_date` and to order the ranking function descending to show most recent purchases prior to becoming a member first.
- `dense_rank` is used once again with regards to lack of context of an "order logged" in this database to what is the actual order of orders placed.
---
**8. What is the total items and amount spent for each member before they became a member?**
```SQL
select
    s.customer_id,
    count(s.product_id) as items_purchased,
    sum(price) as total_spent
    

from dannys_diner.sales as s
inner join dannys_diner.members as mb using(customer_id)
inner join dannys_diner.menu as mn using(product_id)
where order_date < join_date
group by customer_id
order by customer_id;
```
Result:
| customer_id | items_purchased | total_spent | 
| ----------- | ----------- | -- |
| A | 2 | 25 |
| B | 3 | 40 |

- Filtered all tables for orders made before the `join_date` and used aggreate functions to calculate the total items purchased and total spent before customers A and B became members. Ordered by `customer_id`.
---
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```SQL
select
    customer_id,
    sum(case when s.product_id = 1 then price * 20
        else price * 10 end) as total_points

from dannys_diner.sales as s
inner join dannys_diner.menu as m using(product_id)
group by s.customer_id
order by s.customer_id;
```
Result:
| customer_id | total_points |
| ----------- | ----------- |
| A | 860 |
| B | 940 |
| C | 360 |

- Used `case when` to distinguish between regular points and 2x points for sushi.
---
**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```SQL
select
```
---
## Bonus Questions

**Join All The Things**
```SQL
select
```

**Rank All The Things**
```SQL
select
```
