# SQL LeetCode Insights

## Number of Consecutive IDs >= 3

https://leetcode.com/problems/human-traffic-of-stadium/submissions/

```sql
with t1 as(
    select id , visit_date , people,
    id - row_number() OVER(order by id) as grp
    from Stadium
    where people >= 100 
    )
    select id, visit_date, people
    from t1 
    where grp in (select grp from t1 group by grp having count(*) >=3)
```

| iD   |      | ROW_number | id -row_number |
| ---- | ---- | ---------- | -------------- |
| 2    |      | 1          | 1              |
| 3    |      | 2          | 1              |
| 5    |      | 3          | 2              |
| 6    |      | 4          | 2              |
| 7    |      | 5          | 2              |
| 8    |      | 6          | 2              |
| 11   |      | 7          | 4              |
| 12   |      | 8          | 4              |

ROW_NUMBER always increases linearly. ID increases regularly and then has gaps. If you subtract them, all consecutive IDs will get the same number. 

## Self Join on Delete

Write an SQL query to **delete** all the duplicate emails, keeping only one unique email with the smallest `id`. Note that you are supposed to write a `DELETE` statement and not a `SELECT` one.

```sql
DELETE p1
FROM Person p1, Person p2
WHERE p1.Email = p2.Email AND
p1.Id > p2.Id
```

## Second Highest Salary (Subquery)

```sql
select max(salary) SecondHighestSalary 
from Employee
where salary < (select max(salary) from Employee)
```

Does not work for the third, fourth highest salary. 

## Second Highest Salary (Rank)

Trick is to return null in case there is no second highest. Somehow using `MAX` works. 

```sql
select max(salary) SecondHighestSalary
from(
    select salary, 
           RANK() OVER (order by salary desc) as ranked
    from Employee) t1
where ranked > 1
```

This does not work, because it doesn't return `NULL` in case there is no second salary. It returns nothing. 

```sql
select salary SecondHighestSalary
from(
select salary, RANK() OVER (order by salary desc) as ranked
from Employee
    ) t1
    where ranked = 2
```

Using max() will return a NULL if the value doesn't exist. So there is no need to UNION a NULL.

## Employees earning more than their manager

https://leetcode.com/problems/employees-earning-more-than-their-managers/submissions/

```sql
select e1.name as Employee
from Employee e1 inner join Employee e2
on e1.managerId = e2.id 
where e1.salary > e2.salary
```

## Three consecutive numbers

https://leetcode.com/problems/consecutive-numbers/

```sql
select distinct a.num ConsecutiveNums from 
logs a inner join logs b
on a.id = b.id-1
inner join logs c
on b.id = c.id-1
where a.num = b.num and b.num = c.num`
```

```sql
select distinct x.num as ConsecutiveNums
from
(
    select num,
    LEAD(num, 1) over(order by id) as next_num,
    LEAD(num, 2) over(order by id) as next_next_num
    from logs
) x
where x.num = x.next_num 
and x.num = x.next_next_num
```

## MySQL Array to String

https://leetcode.com/problems/group-sold-products-by-the-date/

```sql
select sell_date, 
count(DISTINCT product) as num_sold, 
GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') as products
from Activities
group by sell_date
order by sell_date
```

## Filtering in the JOIN

https://leetcode.com/problems/market-analysis-i/

```sql
SELECT u.user_id AS buyer_id, join_date, COUNT(order_date) AS orders_in_2019 
FROM Users as u
LEFT JOIN Orders as o
ON u.user_id = o.buyer_id
AND YEAR(order_date) = '2019'
GROUP BY u.user_id
```

Filter in the JOIN allows us to basically run a `WHERE` on a single table before joining it. In this case, we all including all users but only counting orders from 2019. If we had placed the condition directly in `WHERE`, we would have deleted all users that didn't make any orders in 2019, which is not what we wanted here. 

## Check for existence in another table (subquery + UNION)

https://leetcode.com/problems/employees-with-missing-information/submissions/

```sql
select employee_id
from Employees
where employee_id not in (select employee_id from Salaries)
union 
select employee_id
from Salaries
where employee_id not in (select employee_id from Employees)
order by employee_id
```

## Unpivot with UNION

https://leetcode.com/problems/rearrange-products-table/

```sql
select product_id, "store1" as store, store1 as price 
from Products
where store1 is not null

union

select product_id, "store2" as store, store2 as price
from Products
where store2 is not null

union
select product_id, "store3" as store, store3 as price
from Products
where store3 is not null

```

## Rising temperature with self join

https://leetcode.com/problems/rising-temperature/

```sql
SELECT w1.Id FROM Weather w1, Weather w2
WHERE subdate(w1.Date, 1) = w2.Date 
AND w1.Temperature > w2.Temperature
```

## Multiple filters / Difference between `ROW_NUMBER()` and `RANK()`

https://leetcode.com/problems/product-sales-analysis-iii/submissions/

Checking for two values at the same time with `WHERE`. 

```sql
select
    product_id,
    year as first_year,
    quantity,
    price
from Sales
where (product_id, year) in (select product_id, min(year) from Sales group by 1)
```

Also, this problem can be solved with a window function, but only if you use `RANK`! In contrast, `ROW_NUMBER()` will fail. Why is that? Because `ROW_NUMBER()` gives a distinct value to every row. So when you filter by `ROW_NUMBER = 1` you are taking only one row. This fails if there are multiple instances of `(product_id, year)` in the `Sales` table. In contrast, `RANK` will give the same value to each of those instances. 

