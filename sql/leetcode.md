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

