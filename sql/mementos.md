# SQL Mementos

## Maxims

It's not a solution if you can't explain it.

## Approaches to consider

When stuck on a problem consider, in order of growing complexity: 

* select
* where
* group by
* join
* self-join
* window functions 

## Joins: min and max resulting rows

https://stackoverflow.com/a/46163627/4540866

https://www.itprotoday.com/sql-server/outer-join-limits

Where L is rows in left table, R is rows in right table. 

|                        | min           | max       | MIN VENN | MAX VENN |
| ---------------------- | ------------- | --------- | -------- | -------- |
| Inner                  | 0             | MIN(L, R) | apart    | subsumed |
| Left Outer             | L             | L + R - 1 | apart    | X        |
| Right Outer            | R             | L + R - 1 | apart    | X        |
| **Full Outer**         | **MAX(L, R)** | **L + R** | subsumed | apart    |
| Cross Join (Cartesian) | L * R         | L * R     |          |          |
| Self Join (left table) | L             | L         |          |          |



## Reserved names	

Never alias a column as `rank` or `max` or any other SQL function.

## Ranks

Know the difference between `DENSE_RANK`, `RANK` and `PERCENT_RANK`. 

Formula for PERCENT_RANK:

```
(rank-1) / ( total_rows-1)  
```

## Distinct

When problem asks for distinct elements, remember to use the `DISTINCT` clause. 

