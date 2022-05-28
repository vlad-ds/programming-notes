# Cleaning Data in Server Databases

You can use `CONCAT`, `FORMAT`, `CAST` and `REPLICATE` to format strings. 

```sql
SELECT 
	-- Concat the strings
	CONCAT(
		carrier_code, 
		' - ', 
      	-- Replicate zeros
		REPLICATE('0', 9 - LEN(registration_code)), 
		registration_code, 
		', ', 
		airport_code)
	AS registration_code
FROM flight_statistics
-- Filter registers with more than 100 delays
WHERE delayed > 100
```

```sql
SELECT 
    -- Concat the strings
	CONCAT(
		carrier_code, 
		' - ', 
        -- Format the code
		FORMAT(CAST(registration_code AS INT), '0000000'),
		', ', 
		airport_code
	) AS registration_code
FROM flight_statistics
-- Filter registers with more than 100 delays
WHERE delayed > 100
```

You can use `TRIM` to remove trailing spaces. Or you can use `RTRIM` and `LTRIM`. 

Use `REPLACE` and `UPPER` to unify string values. Caution: `REPLACE` is case insensitive by default. 

```sql
SELECT 
	airport_code,
	-- Use the appropriate function to remove the extra spaces
    TRIM(airport_name) AS airport_name,
	airport_city,
    airport_state
-- Select the source table
FROM airports
```

```sql
SELECT 
	airport_code, airport_name,
    	-- Convert to uppercase
    	UPPER(
            -- Replace 'Chicago' with 'ch'.
          	REPLACE(airport_city, 'Chicago', 'ch')
        ) AS airport_city,
    airport_state
FROM airports
WHERE airport_code IN ('ORD', 'MDW')
```

You can test string similarity with `SOUNDEX` and `DIFFERENCE`. 

`SOUNDEX` is a phonetic algorithm that returns a four character code expressing the similarity between two strings. 

`DIFFERENCE` compares 2 `SOUNDEX` values and returns a value from 0 to 4. 0 indicates no similarity, 4 indicates identical matching.  

```sql
SELECT 
    -- First name and surname of the statisticians
	DISTINCT S1.statistician_name, S1.statistician_surname
-- Join flight_statistics with itself
FROM flight_statistics S1 INNER JOIN flight_statistics S2 
	-- The SOUNDEX result of the first name and surname have to be the same
	ON SOUNDEX(S1.statistician_name) = SOUNDEX(S2.statistician_name) 
	AND SOUNDEX(S1.statistician_surname) = SOUNDEX(S2.statistician_surname) 
-- The texts of the first name or the texts of the surname have to be different
WHERE S1.statistician_name <> S2.statistician_name
	OR S1.statistician_surname <> S2.statistician_surname
```

```sql
SELECT 
    -- First name and surnames of the statisticians
	DISTINCT S1.statistician_name, S1.statistician_surname
-- Join flight_statistics with itself
FROM flight_statistics S1 INNER JOIN flight_statistics S2 
	-- The DIFFERENCE of the first name and surname has to be equals to 4
	ON DIFFERENCE(S1.statistician_name, S2.statistician_name) = 4
	AND DIFFERENCE(S1.statistician_surname, S2.statistician_surname) = 4
-- The texts of the first name or the texts of the surname have to be different
WHERE S1.statistician_name <> S2.statistician_name
	OR S1.statistician_surname <> S2.statistician_surname
```

---

Missing values are `NULL`. 

`COALESCE` returns the first value in an iterable that is not null. 

```sql
SELECT *
-- Select the appropriate table
FROM airports
-- Exclude the rows where airport_city is NULL
WHERE airport_city IS NOT NULL
```

```sql
SELECT *
-- Select the appropriate table
FROM airports
-- Exclude the rows where airport_city is missing
WHERE airport_city <> ''
```

```sql
SELECT
  airport_code,
  airport_name,
  -- Replace missing values for airport_city with 'Unknown'
  ISNULL(airport_city, 'Unknown') AS airport_city,
  -- Replace missing values for airport_state with 'Unknown'
  ISNULL(airport_state, 'Unknown') AS airport_state
FROM airports
```

```sql
SELECT
airport_code,
airport_name,
-- Replace the missing values
COALESCE(airport_city, airport_state, 'Unknown') AS location
FROM airports
```

Avoiding duplicate data. You can use window functions to number the rows that are equal on a certain number of columns, specifying the ordering. Then you can select only the first row in each group. 

```sql
WITH cte AS (
    SELECT *, 
        ROW_NUMBER() OVER (
            PARTITION BY 
                airport_code, 
                carrier_code, 
                registration_date
			ORDER BY 
                airport_code, 
                carrier_code, 
                registration_date
        ) row_num
    FROM flight_statistics
)
SELECT * FROM cte
-- Exclude duplicates
WHERE row_num = 1;
```

Different date formats. Use `CONVERT` and `FORMAT`. 

```sql
SELECT 
    airport_code,
    carrier_code,
    canceled, 
    airport_code, 
    -- Convert the registration_date to a DATE and print it in mm/dd/yyyy format
    CONVERT(VARCHAR(10), CAST(registration_date AS DATE), 101) AS registration_date
FROM flight_statistics 
-- Convert the registration_date to mm/dd/yyyy format
WHERE CONVERT(VARCHAR(10), CAST(registration_date AS DATE), 101) 
	-- Filter the first six months of 2014 in mm/dd/yyyy format 
	BETWEEN '01/01/2014' AND '06/30/2014'
```

```sql
SELECT 
	pilot_code,
	pilot_name,
	pilot_surname,
	carrier_code,
    -- Convert the entry_date to a DATE and print it in dd/MM/yyyy format
	FORMAT(CAST(entry_date AS DATE), 'dd/MM/yyyy') AS entry_date
from pilots
```

`FORMAT` is not recommended for high volumes of data because it is slower. 

---

Out of range values and inaccurate data. 

```sql
SELECT * FROM series
-- Detect the out of range values
WHERE num_ratings NOT BETWEEN 0 AND 5000
```

```sql
SELECT * FROM series
-- Detect the out of range values
WHERE num_ratings < 0 OR num_ratings > 5000
```

You have to deal with undesirable data types. Use `CAST` and `CONVERT`. 

```sql
-- Use CAST() to convert the num_ratings column
SELECT AVG(CAST(num_ratings AS INT))
FROM series
-- Use CAST() to convert the num_ratings column
WHERE CAST(num_ratings AS INT) BETWEEN 0 AND 5000
```

```sql
-- Use CONVERT() to convert the num_ratings column
SELECT AVG(CONVERT(INT, num_ratings))
FROM series
-- Use CONVERT() to convert the num_ratings column
WHERE CONVERT(INT, num_ratings) BETWEEN 0 AND 5000
```

Pattern matching with `LIKE`. To use actual regex, we need to create and install extensions. `_` indicates one character and `%` indicates zero or more characters. 

```sql
SELECT 
	name,
    -- URL of the official site
	official_site
FROM series
-- Get the URLs that don't match the pattern
WHERE official_site NOT LIKE
	-- Write the pattern
	'www.%'
```

```sql
SELECT 
	name, 
    -- Contact number
    contact_number
FROM series
-- Get the numbers that don't match the pattern
WHERE contact_number NOT LIKE 
	-- Write the pattern
	'555-___-____'
```

---

Concatenating columns, keeping in mind `NULL` values. 

```sql
SELECT 
	client_name,
	client_surname,
    -- Consider the NULL values
	ISNULL(city, '') + ISNULL(', ' + state, '') AS city_state
FROM clients
```

```sql
SELECT 
		client_name,
		client_surname,
    -- Use the function to concatenate the city and the state
		CONCAT(
				city,
				CASE WHEN state IS NULL THEN '' 
				ELSE CONCAT(', ', state) END) AS city_state
FROM clients
```

```sql
SELECT 
	product_name,
	units,
    -- Use the function to concatenate the different parts of the date
	DATEFROMPARTS(
      	year_of_sale, 
      	month_of_sale, 
      	day_of_sale) AS complete_date
FROM paper_shop_daily_sales
```

Splitting a column into more columns. Use `SUBSTRING` and `CHARINDEX`. Use `LEFT`, `RIGHT` and `REVERSE`. 

Example of using `SUBSTRING` in combination with `CHARINDEX`:

```sql
SELECT 
	client_name,
	client_surname,
    -- Extract the name of the city
	SUBSTRING(city_state, 1, CHARINDEX(', ', city_state) - 1) AS city,
    -- Extract the name of the state
    SUBSTRING(city_state, CHARINDEX(', ', city_state) + 1, LEN(city_state)) AS state
FROM clients_split
```

```sql
SELECT
	client_name,
	client_surname,
    -- Extract the name of the city
	LEFT(city_state, CHARINDEX(', ', city_state) - 1) AS city,
    -- Extract the name of the state
    RIGHT(city_state, CHARINDEX(' ,', REVERSE(city_state)) - 1) AS state
FROM clients_split
```

We can transform rows into columns and vice versa. In spreadsheets, pivot tables allow to group data based of a specific set of columns. 

`PIVOT` turns the unique values from one column into multiple columns. To turn columns into rows, use `UNPIVOT`. 

```sql
SELECT
	year_of_sale,
    -- Select the pivoted columns
	notebooks, 
	pencils, 
	crayons
FROM
   (SELECT 
		SUBSTRING(product_name_units, 1, charindex('-', product_name_units)-1) product_name, 
		CAST(SUBSTRING(product_name_units, charindex('-', product_name_units)+1, len(product_name_units)) AS INT) units,	
    	year_of_sale
	FROM paper_shop_monthly_sales) sales
-- Sum the units for column that contains the values that will be column headers
PIVOT (SUM(units) FOR product_name IN (notebooks, pencils, crayons))
-- Give the alias name
AS paper_shop_pivot
```

```sql
SELECT * FROM pivot_sales
-- Use the operator to convert columns into rows
UNPIVOT
	-- The resulting column that will contain the turned columns into rows
	(units FOR product_name IN (notebooks, pencils, crayons))
-- Give the alias name
AS unpivot_sales
```

