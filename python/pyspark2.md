 # Cleaning Data With PySpark

Why PySpark? Problems: optimizing performance, and organizing data flow. Spark is scalable and offers a powerful framework for data handling. 

Spark schema dines the format of a DataFrame. May contain various data types. Can filter garbage data during import. Improves read performance. 

```python
import pyspark.sql.types

peopleSchema = StructType([
	StructField('name', StringType(), True),
	StructField('age', IntegerType(), True),
	StructField('city', StringType(), True)
])

people_df = spark.read.format('csv').load(name='rawdata.csv', schema=peopleSchema)
```

The third Boolean parameter corresponds to whether the field is `nullable`. 

Spark is designed for use with immutable objects. 

Spark uses lazy processing to execute transformations. Transformations are processed only when an action occurs. 

---

Issues with CSV files:

* No defined schema
* Nested data requires special handling
* Encoding format is limtied
* CSV files are slow to parse
* Files cannot be filtered (no "predicate pushdown")
* Intermediate file representations require defining a schema, but this cannot be done until all data is loaded

**Parquet** is a compressed columnar data format. Supported by Spark and data processing frameworks. Supports predicate pushdown and automatically stores schema information. Parquet files are binary. 

```python
df = spark.read.format('parquet').load('filename.parquet')
df = spark.read.parquet('filename.parquet')

df.write.format('parquet').save('filename.parquet')
df.write.parquet('filename.parquet')
```

Parquet can allow us to perform SQL queries. 

```python
flight_df = spark.read.parquet('flights.parquet')
flight_df.createOrReplaceTempView('flights')
short_flights_df = spark.sql(""""SELECT * FROM flights
							 WHERE flightduration < 100
                             """)
```

---

DataFrames :

* Made up of rows and columns
* Immutable
* Support transformation operations

Common transformations: 

* Filter / WHERE
* Select
* withColumn
* drop

Filtering data: remove nulls, remove odd entries, split data from combined sources. You can negate with `~`. 

Spark provides several functions for data transformations. 

```python
import pyspark.sql.functions as F

voter_df.withColumn('upper', F.upper('name'))
voter_df.withColumn('splits', F.split('name', ' '))
voter_df.withColumn('year', voter_df['_c4']).cast(IntegerType())
```

Functions for `ArrayType` columns include `.size(column)`, `getItem(index)`. 

```python
# Show the distinct VOTER_NAME entries
voter_df.select(voter_df['VOTER_NAME']).distinct().show(40, truncate=False)

# Filter voter_df where the VOTER_NAME is 1-20 characters in length
voter_df = voter_df.filter('length(VOTER_NAME) > 0 and length(VOTER_NAME) < 20')

# Filter out voter_df where the VOTER_NAME contains an underscore
voter_df = voter_df.filter(~ F.col('VOTER_NAME').contains('_'))

# Show the distinct VOTER_NAME entries again
voter_df.select('VOTER_NAME').distinct().show(40, truncate=False)
```

 Splitting a name string into first name and last name:

```python
# Add a new column called splits separated on whitespace
voter_df = voter_df.withColumn('splits', F.split(voter_df.VOTER_NAME, '\s+'))

# Create a new column called first_name based on the first item in splits
voter_df = voter_df.withColumn('first_name', voter_df.splits.getItem(0))

# Get the last entry of the splits list and create a column called last_name
voter_df = voter_df.withColumn('last_name', voter_df.splits.getItem(F.size('splits') - 1))

# Drop the splits column
voter_df = voter_df.drop('splits')

# Show the voter_df DataFrame
voter_df.show()
```

Conditional clauses are:

* Inline version of if / then / else
* `.when()`
* `.otherwise()` 

```python
df.select(df.Name, df.Age, 
		  when(df.Age >= 18, "Adult")
		  .when(df.Age < 18, "Minor")
```

```python
# Add a column to voter_df for any voter with the title **Councilmember**
voter_df = voter_df.withColumn('random_val',
                               when(voter_df.TITLE == 'Councilmember', F.rand()))

# Show some of the DataFrame rows, noting whether the when clause worked
voter_df.show()
```

Multiple when clauses and otherwise: 

```python
# Add a column to voter_df for a voter based on their position
voter_df = voter_df.withColumn('random_val',
                               when(voter_df.TITLE == 'Councilmember', F.rand())
                               .when(voter_df.TITLE == 'Mayor', 2)
                               .otherwise(0)
                              )

# Show some of the DataFrame rows
voter_df.show()

# Use the .filter() clause with random_val
voter_df.filter(voter_df.random_val == 0).show()
```

A User defined function or UDF is a Python method. It's defined and wrapped in the `pyspark.sql.functions.udf` method. The result is stored in a variable that can be called like a normal Spark function. 

```python
def reverseString(mystr):
    return mystr[::-1]

#wrap the function and define the return type
udfReverseString = udf(reverseString, StringType())

user_df = user_df.withColumn('ReverseName', 
                            udfReverseString(user_df.Name))
```

```python
def getFirstAndMiddle(names):
  # Return a space separated string of names
  return ' '.join(names[:-1])

# Define the method as a UDF
udfFirstAndMiddle = F.udf(getFirstAndMiddle, StringType())

# Create a new column using your UDF
voter_df = voter_df.withColumn('first_and_middle_name', udfFirstAndMiddle('splits'))

# Show the DataFrame
voter_df.show()
```

Partitioning. Spark breaks dataframes into partitions. Their size varies. Each partition is handled independently. 

Spark has a function that provides monotonically increasing IDs. They are 64 bit Integers that increase in value and are unique, but they're not necessarily sequential. They are divided in groups, covering each partition. 

```python
# Select all the unique council voters
voter_df = df.select(df["VOTER NAME"]).distinct()

# Count the rows in voter_df
print("\nThere are %d rows in the voter_df DataFrame.\n" % voter_df.count())

# Add a ROW_ID
voter_df = voter_df.withColumn('ROW_ID', F.monotonically_increasing_id())

# Show the rows with 10 highest IDs in the set
voter_df.orderBy(voter_df.ROW_ID.desc()).show(10)
```

```python
# Print the number of partitions in each DataFrame
print("\nThere are %d partitions in the voter_df DataFrame.\n" % voter_df.rdd.getNumPartitions())
print("\nThere are %d partitions in the voter_df_single DataFrame.\n" % voter_df_single.rdd.getNumPartitions())

# Add a ROW_ID field to each DataFrame
voter_df = voter_df.withColumn('ROW_ID', F.monotonically_increasing_id())
voter_df_single = voter_df_single.withColumn('ROW_ID', F.monotonically_increasing_id())

# Show the top 10 IDs in each DataFrame 
voter_df.orderBy(voter_df.ROW_ID.desc()).show(10)
voter_df_single.orderBy(voter_df_single.ROW_ID.desc()).show(10)
```

Creating an index with a starting value, so it doesn't overlap with other indexes:

```python
# Determine the highest ROW_ID and save it in previous_max_ID
previous_max_ID = voter_df_march.select('ROW_ID').rdd.max()[0]

# Add a ROW_ID column to voter_df_april starting at the desired value
voter_df_april = voter_df_april.withColumn('ROW_ID', F.monotonically_increasing_id() + previous_max_ID)

# Show the ROW_ID from both DataFrames and compare
voter_df_march.select('ROW_ID').show()
voter_df_april.select('ROW_ID').show()

```

---

**Caching**. Improves speed and reduces resources usage. However, large datasets may not fit in memory. Disk based caching may not be a performance improvement. It only works for repeated tasks. Cache in fast SSD. 

```python
voter_df = spark.read.csv('voter_data.txt.gz')
voter_df.cache().count()
```

Cache works as a transformation. Nothing is actually cached until an action is called. You can use `df.is_cached` to see if a dataframe is cached. To stop caching call `df.unpersist()`. 

How to improve import performance. **Spark Clusters** are made of 2 types of processes:

1. One driver process. Handles assignment and consolidation.
2. As many worker processes as required. Handle transformations and actions. Report back to the driver.

In general it is better to have many smaller import objects than a large one. Spark performs better if objects are of similar size. If you have a slow import, divide your file into many smaller files with approximately the same number of rows. 

A well-defined schema drastically improves import performance. Schemas avoid reading the data multiple times and provide validation on import. 

Spark allows you to import many files via wildcard:

```python
airport_df = spark.read.csv('airports-*.txt.gz')
```

How to split objects? Use OS utilities: 

```bash
split -l 10000 -d largefile chunk-
```

Or use Python scripts. A simple method is to read in a single file and write it back out as Parquet. 

```python
df_csv = spark.read.csv('singlelargefile.csv')
df_csv.write.parquet('data.parquet')
df = spark.read.parquet('data.parquet')
```

Spark contains many **configurations settings**. Can be modified to match specific needs. 

Spark deployment options:

1. Single node
2. Standalone 
3. Managed (Yarn, Mesos, Kubernetes)

Driver: 

* Task assignment
* Monitor processes and tasks 
* Result consolidation 
* Shared data access

Tips: Driver should have double the memory of the worker. Fast local storage is helpful.

Worker:

* Runs actual tasks
* Has all code, data and resources for a given task

Tips: more worker nodes is often better than larger workers. Test to find the correct balance. Fast local storage is extremely useful. 

```python
# Name of the Spark application instance
app_name = spark.conf.get('spark.app.name')

# Driver TCP port
driver_tcp_port = spark.conf.get('spark.driver.port')

# Number of join partitions
num_partitions = spark.conf.get('spark.sql.shuffle.partitions')

# Show the results
print("Name: %s" % app_name)
print("Driver TCP port: %s" % driver_tcp_port)
print("Number of partitions: %s" % num_partitions)
```

The `explain` method on a dataframe. The result is the estimated plan that will be run to generate results from the DataFrame. 

Shuffling. Consists in moving data around to various workers to complete a task. Shuffle is useful and hides complexity from the user. But it can be slow to complete and it lowers the overall throughput of the workers. It's often necessary, but you should try to minimize. 

The `repartition` method reshuffles all the data, which is costly. If you need to change the number of partitions, use `coalesce(num_partitions)`. 

Use care when doing `join`s as they can shuffle data. To avoid this you can use the `broadcast` function. It provides a copy of an object to each worker, preventing undue/excess communication between nodes. 

```python
from pyspark.sql.functions import broadcast
combined_df = df_1.join(broadcast(df_2))
```

---

Data pipeline.  A simple example:

```python
# Import the data to a DataFrame
departures_df = spark.read.csv('2015-departures.csv.gz', header=True)

# Remove any duration of 0
departures_df = departures_df.filter(departures_df[3] > 0)

# Add an ID column
departures_df = departures_df.withColumn('id', F.monotonically_increasing_id())

# Write the file out to JSON format
departures_df.write.json('output.json', mode='overwrite')
```

Spark's CSV parser:

* Removes blank lines
* Can remove comments based on an optional arguments 
* Handles header fields. If a schema is defined, the header is ignored. Otherwise `header = True` will read the headers from the file. 
* Automatically creates columns based on a `sep` argument. Defaults to `,`. 

```python
# Import the file to a DataFrame and perform a row count
annotations_df = spark.read.csv('annotations.csv.gz', sep='|')
full_count = annotations_df.count()

# Count the number of rows beginning with '#'
comment_count = annotations_df.where(col('_c0').startswith('#')).count()

# Import the file to a new DataFrame, without commented rows
no_comments_df = spark.read.csv('annotations.csv.gz', sep='|', comment='#')

# Count the new DataFrame and verify the difference is as expected
no_comments_count = no_comments_df.count()
print("Full count: %d\nComment count: %d\nRemaining count: %d" % (full_count, comment_count, no_comments_count))
```

Splitting a string into several columns: 

```python
# Split the content of _c0 on the tab character (aka, '\t')
split_cols = F.split(annotations_df['_c0'], '\t')

# Add the columns folder, filename, width, and height
split_df = annotations_df.withColumn('folder', split_cols.getItem(0))
split_df = split_df.withColumn('filename', split_cols.getItem(1))
split_df = split_df.withColumn('width', split_cols.getItem(2))
split_df = split_df.withColumn('height', split_cols.getItem(3))

# Add split_cols as a column
split_df = split_df.withColumn('split_cols', split_cols)
```

Validation is verifying that a dataset complies with the expected format. This involves checking the number of rows/columns and the data types. 

Validating via joins compares data against known values and is comparatively fast. 

You can validate through computations or by verifying against an external source. You might have to use a UDF to modify/verify the dataframe. 

**Analysis calculations** consist in using UDFs for complex analyses.

But UDFs are not the best performing. We should try to execute inline calculations whenever possible. 