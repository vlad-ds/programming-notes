# PySpark

Spark is written in Scala but available in other languages. At the center of Spark is the Apache Spark Core. The rest of the libraries are built on top of it by accessing the RDD API. These libraries are: Spark SQL, MLlib Machine Learning, GraphX and Spark Streaming. 

Spark can be deployed in Local mode or in Cluster mode. Usually you start in Local for development and then move to Cluster for production. No code change is necessary in the transition from Local to Cluster. 

Spark works in cluster to split tasks and parallelize computations. The cluster is hosted on a remote machine called *master* that splits up the data. The master is connected to the other computers, which are *workers*. 

PySpark is the Python API for Spark. 

To create a connection you instantiate the `SparkContext` class. 

Spark's core structure is the Resilient Distributed Dataset (RDD). This is usually accessed through a Spark DataFrame abstraction. To work with Spark DataFrames, you create a `SparkSession` object from the `SparkContext`. The latter is the connection to the cluster and the former is the interface with that connection. 

```python
# Import SparkSession from pyspark.sql
from pyspark.sql import SparkSession

# Create my_spark
spark = SparkSession.builder.getOrCreate()

# Print the tables in the catalog
spark.catalog.listTables()

# Access table with SQL query
query = "FROM flights SELECT * LIMIT 10"
flights10 = spark.sql(query)
flights10.show()

# Convert to Pandas DataFrame
flights_df = flights10.toPandas()

```

Spark provides a `.createDataFrame()` method that takes a `pandas` DataFrame and returns a Spark DataFrame. The output is stored locally. To access it, you can save it as a *temporary table* with the `.createTempView()`method. It registers it as a table in the catalog, but only in this `SparkSession`. The method `.createOrReplaceTempView()`updates the table if already defined. 

```python
# Create pd_temp
pd_temp = pd.DataFrame(np.random.random(10))

# Create spark_temp from pd_temp
spark_temp = spark.createDataFrame(pd_temp)

# Examine the tables in the catalog
print(spark.catalog.listTables())

# Add spark_temp to the catalog
spark_temp.createOrReplaceTempView("temp")

# Examine the tables in the catalog again
print(spark.catalog.listTables())
```

You can also load files directly in Spark:

```python
file_path = "/usr/local/share/datasets/airports.csv"
airports = spark.read.csv(file_path, header=True)
airports.show()
```

You can't directly update a Spark DataFrame because it is immutable. All update methods return a new DataFrame which must be reassigned to the variable:

```python
df = df.withColumn("newCol", df.oldCol + 1)
```

The `.filter()` method is the Spark equivalent of SQL's `WHERE` clause. It takes either a `WHERE` SQL expression as a string, or a Spark Column of boolean values. 

```python
flights.fliter("air_time > 120").show()
flights.filter(flights.air_time > 120).show()
```

In contrast to `.withColumn()`, `.select()` returns only the columns that you specify. Column names can be passed as strings or column objects. 

```python
# Select the first set of columns
selected1 = flights.select("tailnum", "origin", "dest")

# Select the second set of columns
temp = flights.select(flights.origin, flights.dest, flights.carrier)

# Define first filter
filterA = flights.origin == "SEA"

# Define second filter
filterB = flights.dest == "PDX"

# Filter the data, first by filterA then by filterB
selected2 = temp.filter(filterA).filter(filterB)
```

You can also perform operations on the fly and apply aliases.

```python
flights.select((flights.air_time/60).alias("duration_hrs"))
flights.selectExpr("air_time/60 as duration_hrs")

#select multiple columns with SQL
speed2 = flights.selectExpr("origin", "dest", "tailnum", "distance/(air_time/60) as avg_speed")
```

To find the minimum value of `col`:

```python
df.groupBy().min("col").show() #returns a DataFrame
```

```python
# Find the shortest flight from PDX in terms of distance
flights.filter(flights.origin == 'PDX').groupBy().min("distance").show()

# Find the longest flight from SEA in terms of air time
flights.filter(flights.origin == 'SEA').groupBy().max("air_time").show()
```

Using `GroupedData` objects: 

```python
# Group by tailnum
by_plane = flights.groupBy("tailnum")

# Number of flights each plane made
by_plane.count().show()

# Group by origin
by_origin = flights.groupBy("origin")

# Average duration of flights from PDX and SEA
by_origin.avg("air_time").show()
```

The submodule `pyspark.sql.functions` contains useful aggregation functions. They all take the name of a column in a `GroupedData` table. 

```python
import pyspark.sql.functions as F

# Group by month and dest
by_month_dest = flights.groupBy('month', 'dest')
# Average departure delay by month and destination
by_month_dest.avg("dep_delay").show()
# Standard deviation of departure delay
by_month_dest.agg(F.stddev("dep_delay")).show()
```

To perform joins: 

```python
# Examine the data
print(airports.show())

# Rename the faa column
airports = airports.withColumnRenamed("faa", "dest")

# Join the DataFrames
flights_with_airports = flights.join(airports, on="dest", how="leftouter")

# Examine the new DataFrame
print(flights_with_airports.show())
```

## ML pipelines

`pyspark.ml` includes the classes `Transformer` and `Estimator`. 

`Transformer` classes have a `.transform()` method that takes a DataFrame and returns a new DataFrame. `Estimator` classes implement a `.fit()` method. They take a DataFrame and return a model object. 

Spark only handles numeric data. The `.cast()` method can be used to convert columns. 

```python
dataframe = dataframe.withColumn("col", dataframe.col.cast("new_type"))
```

To define a new column as a combination of two columns: 

```python
model_data = model_data.withColumn("plane_age", model_data.year - model_data.plane_year)
```

Spark does not accept strings. To convert categorical variables:

* Create a `StringIndexer`. This is an `Estimator` that takes a DataFrame with a column of strings and maps each unique string to a number. It returns a `Transformer` that takes a DataFrame, attaches the mapping to it as metadata, and returns a new DataFrame with the encoded column. 
* Encode this numeric column as a one-hot vector using a `OneHotEncoder`. 

```python
#encode the carrier column

# Create a StringIndexer
carr_indexer = StringIndexer(inputCol="carrier", outputCol="carrier_index")

# Create a OneHotEncoder
carr_encoder = OneHotEncoder(inputCol="carrier_index", outputCol="carrier_fact")
```

For machine learning, all the columns need to be combined in one vector column through the `VectorAssembler`. 

```python
# Make a VectorAssembler
vec_assembler = VectorAssembler(inputCols=["month", "air_time", "carrier_fact", "dest_fact", "plane_age"], outputCol="features")
```

`Pipeline` is a class in the `pyspark.ml` module that combines all the`Estimators` and `Transformers`. 

```python
# Import Pipeline
from pyspark.ml import Pipeline

# Make the pipeline
flights_pipe = Pipeline(stages=[dest_indexer, dest_encoder, carr_indexer, carr_encoder, vec_assembler])
```

In Spark you must split the data after all the transformations. That is because the indexers are not consistent. 

```python
# Fit and transform the data
piped_data = flights_pipe.fit(model_data).transform(model_data)
# Split the data into training and test sets
training, test = piped_data.randomSplit([.6, .4])
# Import LogisticRegression
from pyspark.ml.classification import LogisticRegression
# Create a LogisticRegression Estimator
lr = LogisticRegression()
# Import the evaluation submodule
import pyspark.ml.evaluation as evals
# Create a BinaryClassificationEvaluator
evaluator = evals.BinaryClassificationEvaluator(metricName="areaUnderROC")
```

```python
# Import the tuning submodule
import pyspark.ml.tuning as tune
# Create the parameter grid
grid = tune.ParamGridBuilder()
# Add the hyperparameter
grid = grid.addGrid(lr.regParam, np.arange(0, .1, .01))
grid = grid.addGrid(lr.elasticNetParam, [0, 1])
# Build the grid
grid = grid.build()
# Create the CrossValidator
cv = tune.CrossValidator(estimator=lr,
                         estimatorParamMaps=grid,
                         evaluator=evaluator)
```

```python
# Fit cross validation models
models = cv.fit(training)
# Extract the best model
best_lr = models.bestModel
# Use the model to predict the test set
test_results = best_lr.transform(test)
# Evaluate the predictions
print(evaluator.evaluate(test_results))
```

----

# A PySpark Pipeline

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
#only a reference to the table is loaded
prices = spark.read.options(header='true').csv(filepath)
```

Remember to manually assign the right data types to the columns!

```python
schema = StructType([StructField("store", StringType(), nullable=False),
                    ...])
prices = spark.read.options(header='true').schema(schema).csv(filepath)
```

Conditionally replace values in data processing: 

```python
from pyspark.sql.functions import col, when
from datetime import date, timedelta
one_year_from_now = date.today().replace(year=date.today().year + 1)
better_frame = employees.withColumn("end_date",
                                   when(col("end_date") > one_year_from_now, None)
                                   .otherwise(col("end_date")))
better_frame.show()
```

Select:

```python
from pyspark.sql.functions import col

# Select the columns and rename the "absorption_rate" column
result = ratings.select([col("brand"),
                       col("model"),
                       col("absorption_rate").alias('absorbency')])

# Show only unique values
result.distinct().show()
```

Aggregate:

```python
from pyspark.sql.functions import col, avg, stddev_samp, max as sfmax

aggregated = (purchased
              # Group rows by 'Country'
              .groupBy(col('Country'))
              .agg(
                # Calculate the average salary per group and rename
                avg('Salary').alias('average_salary'),
                # Calculate the standard deviation per group
                stddev_samp('Salary'),
                # Retain the highest salary per group and rename
                sfmax('Salary').alias('highest_salary')
              )
             )

aggregated.show()
```

To run Spark you use `spark-submit`. It sets up launch environment for use with the *cluster manager* and the selected *deploy mode*. 

---

## Testing

To provide unit testing for our pipelines, we construct DataFrames in memory. For testing, it is best to have small, reusable and well-named functions. Each transformation can be tested and reused. 

```python
from datetime import date
from pyspark.sql import Row

Record = Row("country", "utm_campaign", "airtime_in_minutes", "start_date", "end_date")

# Create a tuple of records
data = (
  Record("USA", "DiapersFirst", 28, date(2017, 1, 20), date(2017, 1, 27)),
  Record("Germany", "WindelKind", 31, date(2017, 1, 25), None),
  Record("India", "CloseToCloth", 32, date(2017, 1, 25), date(2017, 2, 2))
)

# Create a DataFrame from these records
frame = spark.createDataFrame(data)
frame.show()
```

The best known Python modules for tests are `unittest`, `doctest`, `pytest`, `nose`. The first 2 are in stdlib. Their core task is to `assert` something. 

CI/CD:

* **Continuous Integration**. Get code changes integrated with the master branch regularly. Only if tests pass.
* **Continuous delivery**. All artifacts should be in a deployable state at any time without any problem. 

CircleCI is a service that runs tests. It looks for `.circleci/config.yml`. The workflow is: checkout code, install test & build requirements, run tests, package/build the software artefacts. 

----

PySpark shell. Python CLI that allows interface with Spark data structures. Supports connection to a cluster. 

`SparkContext` is an entry point, where control is transferred from the OS to the program. You can access it in the PySpark shell as `sc`. 

```bash
sc.version 		#SparkContext version
sc.pythonVer 	#Python version
sc.master 		#URL of the cluster (or "local" to run in local)
```

Two ways to load data in PySpark: 

```python
rdd1 = sc.parallelize([1, 2, 3, 4, 5], minPartitions=6)
rdd2 = sc.textFile("test.txt", minPartitions=6)
rdd2.getNumPartitions() #get the number of partitions
```

**RDDs** are Spark's core abstraction for working with data. It stands for **Resilient Distributed Datasets**. It is an immutable collection of data distributed across the cluster. It is the fundamental and backbone data type in Spark. 

Given a data file on disk, Spark driver creates an RDD and distributes data among nodes. 

* Resilient. Ability to withstand failures. 
* Distributed. Jobs are distributed across multiple nodes for efficient computation. 
* Datasets. A collection of partitioned data (Arrays, Tables, Tuples).

You can create an RDD by parallelizing an existing collection of objects or by loading data from external datasets (files in HDFS, Objects in Amazon S3, or lines in a text file). They can also be created from existing RDDs. 

RDDs support two types of operations: **transformations** and **actions**. Transformations create new RDDs. Actions perform computations on the RDD. Transformations use **lazy evaluation**. Execution of the transformation graph happens only when an action is performed on RDD. 

Basic RDD operations are `map`, `filter`, `flatMap` and `union`. `map` applies a function to every element. `filter` selects elements based on a boolean function. `flatMap` works like map but returns multiple elements for every input (e.g. splitting a string). `union` returns the union between RDDs. 

Actions return a value after running a computation on the RDD. Basic actions are `collect`, `take(n)`, `first` and `count`.  

```python
# Filter the fileRDD to select lines with Spark keyword
fileRDD_filter = fileRDD.filter(lambda line: 'Spark' in line)

# How many lines are there in fileRDD?
print("The total number of lines with the keyword Spark is", fileRDD_filter.count())

# Print the first four lines of fileRDD
for line in fileRDD_filter.take(4): 
  print(line)
```

Pair RDD is a special data structure to work with datasets where each row is a key that maps to one or more values. They can be created from a list of key-value tuples or from a regular RDD. 

```python
my_tuple = [('Sam', 23), ('Mary', 34), ('Peter', 25)]
pairRDD_tuple = sc.parallelize(my_tuple)

my_list = ['Sam 23', 'Mary 34', 'Peter 25']
regularRDD = sc.parallelize(my_list)
pairRDD_RDD = regularRDD.map(lambda s: (s.split(' ')[0], s.split(' ')[1]))

```

All regular transformations work on RDDs but you have to pass functions that operate on key value pairs. Typical transformations are `reduceByKey`, `groupByKey`, `sortByKey` and `join`. 

```python
# Create PairRDD Rdd with key value pairs
Rdd = sc.parallelize([(1,2),(3,4),(3,6),(4,5)])

# Apply reduceByKey() operation on Rdd
Rdd_Reduced = Rdd.reduceByKey(lambda x, y: x+y)

# Iterate over the result and print the output
for num in Rdd_Reduced.collect(): 
  print("Key {} has {} Counts".format(num[0], num[1]))
```

```python
# Sort the reduced RDD with the key by descending order
Rdd_Reduced_Sort = Rdd_Reduced.SortByKey(ascending=False)

# Iterate over the result and print the output
for num in Rdd_Reduced_Sort.collect():
  print("Key {} has {} Counts".format(num[0], num[1]))
```

```python
# Sort the reduced RDD with the key by descending order
Rdd_Reduced_Sort = Rdd_Reduced.sortByKey(ascending=False)

# Iterate over the result and print the output
for num in Rdd_Reduced_Sort.collect():
  print("Key {} has {} Counts".format(num[0], num[1]))
```

Advanced RDD actions. `reduce(func)` is used for aggregating the elements of a regular RDD. To be computed in parallel, the function should be commutative and associative. 

```python
#summing up all elements in RDD with reduce
x = [1, 3, 4, 6]
RDD = sc.parallelize(x)
RDD.reduce(lambda x, y : x +y)
```

You can recompose the RDD into a single file and save it as a text file.

```python
RDD.coalesce(1).saveAsTextFile("tempFile")
```

RDD actions include `countByKey` and `collectAsMap`. 

```python
rdd = sc.parallelize([("a", 1), ("b", 1), ("a", 1)])
for kee, val in rdd.countByKey().items():
	print(kee, val)
```

`collectAsMap` returns the key-value pairs in the RDD as a dictionary. This operation loads all the data into memory. 

Example of a task: calculating the most common words in the Complete Works of Shakespeare. 

```python
# Create a baseRDD from the file path
baseRDD = sc.textFile(file_path)

# Split the lines of baseRDD into words
splitRDD = baseRDD.flatMap(lambda x: x.split(' '))

# Count the total number of words
print("Total number of words in splitRDD:", splitRDD.count())

# Convert the words in lower case and remove stop words from stop_words
splitRDD_no_stop = splitRDD.filter(lambda x: x.lower() not in stop_words)

# Create a tuple of the word and 1 
splitRDD_no_stop_words = splitRDD_no_stop.map(lambda w: (w, 1))

# Count of the number of occurences of each word
resultRDD = splitRDD_no_stop_words.reduceByKey(lambda x, y: x + y)

# Display the first 10 words and their frequencies
for word in resultRDD.take(10):
	print(word)

# Swap the keys and values 
resultRDD_swap = resultRDD.map(lambda x: (x[1], x[0]))

# Sort the keys in descending order
resultRDD_swap_sort = resultRDD_swap.sortByKey(ascending=False)

# Show the top 10 most frequent words and their frequencies
for word in resultRDD_swap_sort.take(10):
	print("{} has {} counts". format(word[1], word[0]))
```

---

## PySpark SQL

Main abstraction is PySpark DataFrame. An immutable distributed collection of data with named columns. Designed for both structured and semi-structured data. DataFrames support both SQL queries and expression methods. 

`SparkSession` provides a single point of entry to intreact with Spark DataFrames. It's available in shell as `spark`. 

DataFrames can be created from existing RDDs with the `createDataFrame()` method and from various data sources with the `read` method. 

`Schema` defines the structure of data and helps Spark to optimize queries on the data. 

```python
# Create a list of tuples
sample_list = [('Mona',20), ('Jennifer',34), ('John',20), ('Jim',26)]

# Create a RDD from the list
rdd = sc.parallelize(sample_list)

# Create a PySpark DataFrame
names_df = spark.createDataFrame(rdd, schema=['Name', 'Age'])

# Check the type of names_df
print("The type of names_df is", type(names_df))
```

```python
# Create an DataFrame from file_path
people_df = spark.read.csv(file_path, header=True, inferSchema=True)

# Check the type of people_df
print("The type of people_df is", type(people_df))
```

Common Transformations: `select`, `filter`, `groupby`, `orderby`, `dropDuplicates`, `withColumnRenamed`. 

Common Actions: `printSchema`, `head`, `show`, `count`, `columns`, `describe`. 

You can interact through the DataFrame API or through SQL queries. API calls are easier to construct programmatically. SQL queries are concise, easier to understand and highly portable. 

```python
#to use SQL, we create a temporary view of our dataframe
df.createOrReplaceTempView("table1")
df2 = spark.sql("SELECT field1, field2 FROM table1")
df2.collect()
```

Visualization. Is done with three methods: `pyspark_dist_explore` library, `toPandas()`, `HandySpark` library. 

`pyspark_dist_explore` only has three methods: `hist`, `distplot`, `pandas_histogram`. 

Differences between Pandas DataFrame vs Pyspark DataFrame:

1. Pandas DFs are in memory, single-server. Operations on PySpark run in parallel. 
2. Operations in PySpark are lazy evaluation. 
3. PySpark DFs are immutable. 
4. Pandas API supports more operations.

---

## PySpark MLlib

Library specialized in machine learning. Provides ML algorithms, featurization, pipelines. It only supports `RDD`. 

Contains data dypes `Vectors` and `LabeledPoint`. Vectors can be sparse or dense. `LabeledPoint` includes features and predicted value. 

Collaborative filtering example: 

```python
# Load the data into RDD
data = sc.textFile(file_path)

# Split the RDD 
ratings = data.map(lambda l: l.split(','))

# Transform the ratings RDD 
ratings_final = ratings.map(lambda line: Rating(int(line[0]), int(line[1]), float(line[2])))

# Split the data into training and test
training_data, test_data = ratings_final.randomSplit([0.8, 0.2])
```

```python
# Create the ALS model on the training data
model = ALS.train(training_data, rank=10, iterations=10)

# Drop the ratings column 
testdata_no_rating = test_data.map(lambda p: (p[0], p[1]))

# Predict the model  
predictions = model.predictAll(testdata_no_rating)

# Print the first rows of the RDD
predictions.take(2)
```

```python
# Prepare ratings data
rates = ratings_final.map(lambda r: ((r[0], r[1]), r[2]))

# Prepare predictions data
preds = predictions.map(lambda r: ((r[0], r[1]), r[2]))

# Join the ratings data with predictions data
rates_and_preds = rates.join(preds)

# Calculate and print MSE
MSE = rates_and_preds.map(lambda r: (r[1][0] - r[1][1])**2).mean()
print("Mean Squared Error of the model for the test data = {:.2f}".format(MSE))
```

Classification example: 

