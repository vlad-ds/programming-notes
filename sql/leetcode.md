# SQL LeetCode Insights

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
