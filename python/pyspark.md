# PySpark

Spark works in cluster to split tasks and parallelize computations. The cluster is hosted on a remote machine called *master* that splits up the data. The master is connected to the other computers, which are *workers*. 

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

* Create a `StringIndexer`. This is an `Estimator` that takes a DataFrame with a column of strings and maps each unique string to a number. It retuns a `Transformer` that takes a DataFrame, attaches the mapping to it as metadata, and returns a new DataFrame with the encoded column. 
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