# Spark SQL in Python

Spark can act as a distributed SQL query engine. 

Load the data into a dataframe and create a temporary view. 

```python
# Load trainsched.txt
df = spark.read.csv("trainsched.txt", header=True)

# Create temporary table called table1
df.createOrReplaceTempView('table1')

# Inspect the columns in the table df
spark.sql("DESCRIBE schedule").show()
```

Window Functions. When processing rows, each row uses the values of other rows to calculate its value. 

Using a WF to shift the row by 1 (and select next value):

```sql
SELECT train_id, station, time
LEAD(time, 1) OVER (PARTITION BY train_id ORDER BY time) AS time_next
FROM sched
```

Window functions allow you to compute aggregated values over all the rows until the row in question. 

Aggregation can be done through SQL or through dot notation. 

```python
# Give the identical result in each command
spark.sql('SELECT train_id, MIN(time) AS start FROM schedule GROUP BY train_id').show()
df.groupBy('train_id').agg({'time':'min'}).withColumnRenamed('min(time)', 'start').show()

# Print the second column of the result
spark.sql('SELECT train_id, MIN(time), MAX(time) FROM schedule GROUP BY train_id').show()
result = df.groupBy('train_id').agg({'time':'min', 'time':'max'})
result.show()
print(result.columns[1])
```

---

## NLP analysis with Spark. 

```python
# Load the dataframe
df = spark.read.load('sherlock_sentences.parquet')

# Filter and show the first 5 rows
df.where('id > 70').show(5, truncate=False)
```

```python
# Split the clause column into a column called words 
split_df = clauses_df.select(split('clause', ' ').alias('words'))
split_df.show(5, truncate=False)

# Explode the words column into a column called word 
exploded_df = split_df.select(explode('words').alias('word'))
exploded_df.show(10)

# Count the resulting number of rows in exploded_df
print("\nNumber of rows: ", exploded_df.count())
```

Suppose that you know that the upcoming processing steps are going to be grouping the data on chapters. Processing the data will be most efficient if each chapter stays within a single machine. To avoid unnecessary shuffling of the data from one machine to another, let's repartition the dataframe into one partition per chapter.

```python
# Repartition text_df into 12 partitions on 'chapter' column
repart_df = text_df.repartition(12, 'chapter')

# Prove that repart_df has 12 partitions
repart_df.rdd.getNumPartitions()
```

```python
# Find the top 10 sequences of five words
query = """
SELECT w1, w2, w3, w4, w5, COUNT(*) AS count FROM (
   SELECT word AS w1,
   LEAD(word, 1) OVER(PARTITION BY part ORDER BY id) AS w2,
   LEAD(word, 2) OVER(PARTITION BY part ORDER BY id) AS w3,
   LEAD(word, 3) OVER(PARTITION BY part ORDER BY id) AS w4,
   LEAD(word, 4) OVER(PARTITION BY part ORDER BY id) AS w5
   FROM text
)
GROUP BY w1, w2, w3, w4, w5
ORDER BY count DESC
LIMIT 10 """
df = spark.sql(query)
df.show()
```

You can also encase subqueries in a complex way:

```python
sobquery = '''SOME SQL'''

query = """
SELECT chapter, w1, w2, w3, count FROM
(
  SELECT
  chapter,
  ROW_NUMBER() OVER (PARTITION BY chapter ORDER BY count DESC) AS row,
  w1, w2, w3, count
  FROM ( %s )
)
WHERE row = 1
ORDER BY chapter ASC
""" % subquery

spark.sql(query).show()
```

Caching. Spark is aggressive about freeing up memory. Spark's memory policy is LRU (Least Recently Used). Eviction happens independently on each worker.

```python
#cache
df.cache()
#uncache
df.unpersist()
#check
df.is_cached
```

```python
# Unpersist df1 and df2 and initializes a timer
prep(df1, df2) 

# Persist df2 using memory and disk storage level 
df2.persist(storageLevel=pyspark.StorageLevel.MEMORY_AND_DISK)

# Run actions both dataframes
run(df1, "df1_1st") 
run(df1, "df1_2nd")
run(df2, "df2_1st")
run(df2, "df2_2nd", elapsed=True)
```

Caching with tables:

```python
# List the tables
print("Tables:\n", spark.catalog.listTables())

# Cache table1 and Confirm that it is cached
spark.catalog.cacheTable('table1')
print("table1 is cached: ", spark.catalog.isCached('table1'))

# Uncache table1 and confirm that it is uncached
spark.catalog.uncacheTable('table1')
print("table1 is cached: ", spark.catalog.isCached('table1'))
```

---

**Spark Task** is a unit of execution that runs on a single CPU. 

**Spark Stage** is a group of tasks that perform the same computation in parallel, each task typically running a different subset of the data. 

**Spark Job** is a computation triggered by an **action**, sliced into one or more stages. 

Spark UI  gives an overview of all the operations. 

Spark supports logging. But there is a risk of stealth CPU wastage. Methods which trigger database actions should be disabled as far as possible in production so they are not silently executed. 

```python
import logging
logging.basicConfig(stream=sys.stdout, level=logging.DEBUG,
                    format='%(levelname)s - %(message)s')
```

```python
# Log columns of text_df as debug message
logging.debug("text_df columns: %s", text_df.columns)

# Log whether table1 is cached as info message
logging.info("table1 is cached: %s", spark.catalog.isCached(tableName="table1"))

# Log first row of text_df as warning message
logging.warning("The first row of text_df:\n %s", text_df.first())

# Log selected columns of text_df as error message
logging.error("Selected columns: %s", text_df.select("id", "word"))
```

Query plans. In SQL it is possible to obtain a query plan, which is a string detailing the steps entailed by the query.

```sql
EXPLAIN SELECT * FROM table1
```

In Spark: 

```python
# Run explain on text_df
text_df.explain()

# Run explain on "SELECT COUNT(*) AS count FROM table1" 
spark.sql("SELECT COUNT(*) AS count FROM table1").explain()

# Run explain on "SELECT COUNT(DISTINCT word) AS words FROM table1"
spark.sql("SELECT COUNT(DISTINCT word) AS words FROM table1").explain()
```

---

**ETS**. Extract, Transform, Select. 







