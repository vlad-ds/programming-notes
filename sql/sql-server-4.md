# Improving Query Performance in SQL server

Whatever you do in `WHERE` will run on every row. Avoid using calculations and functions in `WHERE`. Use subqueries when necessary. 

`HAVING` runs after a `GROUP BY`. We don't want to use `HAVING` instead of `WHERE` (even though we could) because `WHERE` will filter out rows before grouping and further processing. Don't use `HAVING` to filter individual or ungrouped rows. 

Processing order after select: `DISTINCT`, `ORDER BY`, and finally row limiters such as `TOP`, `ROWNUM` and `LIMIT`.  

`SELECT *` is bad for performance. It selects all columns. In joins, it returns duplicates of joining columns. Always explicitly state the columns you need. 

```sql
SELECT TOP 25 PERCENT -- Limit rows to the upper quartile
       Latitude,
       Longitude,
	   Magnitude,
	   Depth,
	   NearestPop
FROM Earthquakes
WHERE Country = 'PG'
	OR Country = 'ID'
ORDER BY Magnitude DESC; -- Order the results
```

`UNION`, the SQL command that appends tables, should remove duplicates. `UNION ALL` will not remove duplicate rows. 

---

Applications of sub-queries:

* In `FROM`, where the subquery acts as a virtual table or data source.
* In `WHERE`, to return a filter condition to the outer query. 
* In `SELECT`, to derive a new column. 

Sub-queries can be:

* Uncorrelated. Do not contain a reference to the outer query. Can run independently of the outer query. Typically in `WHERE` and `FROM`.
* Correlated. Typically in `WHERE` and `SELECT`.

Correlated sub-queries are inefficient because they run for each row of the outer query. Uncorrelated sub-queries are executed only once. Often the results of a correlated sub-query can be replicated with an `INNER JOIN`. 

Uncorrelated query:

```sql
SELECT UNStatisticalRegion,
       CountryName 
FROM Nations
WHERE Code2 -- Country code for outer query 
         IN (SELECT Country -- Country code for sub-query
             FROM Earthquakes
             WHERE depth >= 400 ) -- Depth filter
ORDER BY UNStatisticalRegion;
```

Correlated query:

```sql
SELECT UNContinentRegion,
       CountryName, 
        (SELECT AVG(magnitude) -- Add average magnitude
        FROM Earthquakes e 
         	  -- Add country code reference
        WHERE n.Code2 = e.Country) AS AverageMagnitude 
FROM Nations n
ORDER BY UNContinentRegion DESC, 
         AverageMagnitude DESC;
```

Substituting correlated query for `INNER JOIN`: 

```sql
SELECT n.CountryName, 
       c.BiggestCity 
FROM Nations AS n
INNER JOIN -- Join the Nations table and sub-query
    (SELECT CountryCode, 
     MAX(Pop2017) AS BiggestCity 
     FROM Cities
     GROUP BY CountryCode) AS c
ON n.Code2 = c.CountryCode; -- Add the joining columns
```

Sometimes we need to check for presence or absence of data. 

We can combine tables with `INTERSECT` and `EXCEPT`. They are great for interrogation and remove duplicates, but the number and order of columns in the `SELECT` statement must be the same between queries. 

```sql
SELECT Capital
FROM Nations -- Table with capital cities

INTERSECT -- Add the operator to compare the two queries

SELECT NearestPop -- Add the city name column
FROM Earthquakes;
```

```sql
SELECT Code2 -- Add the country code column
FROM Nations

EXCEPT -- Add the operator to compare the two queries

SELECT Country 
FROM Earthquakes; -- Table with country codes
```

You can also use `EXISTS`, `NOT EXISTS`, `IN` and `NOT IN`. The advantage is that the results can contain any column from the outer query and in any order. 

`EXISTS` stops processing the sub-query when the condition is `TRUE`. `IN` always processes all the sub-query. Try using `EXISTS` with a sub-query. 

When using `NOT IN`, you need to apply a `IS NOT NULL` filter. `NOT EXISTS` does not have this issue. 

```sql
-- Second attempt
SELECT CountryName,   
	   Capital,
       Pop2016, -- 2016 country population
       WorldBankRegion
FROM Nations AS n
WHERE EXISTS -- Add the operator to compare queries
	  (SELECT 1
	   FROM Earthquakes AS e
	   WHERE n.Capital = e.NearestPop); -- Columns being compared
```

```sql
SELECT WorldBankRegion,
       CountryName,
	   Code2,
       Capital, -- Country capital column
	   Pop2017
FROM Nations AS n
WHERE NOT EXISTS -- Add the operator to compare queries
	(SELECT 1
	 FROM Cities AS c
	 WHERE n.Code2 = c.CountryCode); -- Columns being compared
```

Another way is to use `INNER JOIN` or `LEFT OUTER JOIN`. 

-------

