# Recap
So clearly, I was not able to carry the enthusiasm too far along. Because of various random reasons, I fell offtrack, although it's not that I did not do anything. Just nothing worthwhile.
Anyway, I have started on the ML with Apache Spark course on Coursera and will resume that here.

# Env Setup
Was planning an env setup but now I'm thinking I'll skip and use the setup provided by Coursera instead to save time. Although ironically, the first lab that I'm going to do is about Spark Cluster Setup


# Course Labs
## M2 Labs
### 1. Connecting to Spark Cluster
- Need to install PySpark and FindSpark. This is when the cluster is already setup.
  
```python
# FindSpark simplifies the process of using Apache Spark with Python

import findspark
findspark.init()

# import SparkSession
from pyspark.sql import SparkSession
```

#### Creating Spark Session and Load Data
```python
## Create Spark Session using given name
spark = SparkSession.builder.appName("Getting Started with Spark").getOrCreate()

## Load dataset
mpg_data = spark.read.csv("mpg.csv", header=True, inferSchema=True)

## Print Schema
mpg_data.printSchema()

## See data snapshot
mpg_data.head(3)

## Stopping Spark Session
spark.stop()
```

```
root
 |-- MPG: double (nullable = true)
 |-- Cylinders: integer (nullable = true)
 |-- Engine Disp: double (nullable = true)
 |-- Horsepower: integer (nullable = true)
 |-- Weight: integer (nullable = true)
 |-- Accelerate: double (nullable = true)
 |-- Year: integer (nullable = true)
 |-- Origin: string (nullable = true)
```

### 2. Distributed Architecture of Spark
- `SparkContext` is the traditional entry point for a Spark application. It's the "connection" to the Spark cluster and is responsible for creating and managing **Resilient Distributed Datasets (RDDs)**, which are Spark's fundamental, low-level data structure.
- `SparkSession` is the modern, unified entry point for all Spark functionality, while `SparkContext` was the original entry point, primarily for low-level RDD operations.
- Now the notebooks from the course uses `SparkContext`, but since `SparkSession` is the modern way, I'll try to accomplish everything using that only. I expect the code to break in places, but hopefully that shan't stop me.

```bash
# Data URL
https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/XXlNzqYcxqkTbllc-tL_0w/Retailsales.csv
```

```python
from pyspark.sql import SparkSession
spark_sess = SparkSession.builder.appName("RetailSalesAnalysis").getOrCreate()
retail_df = spark_sess.read.csv("Retailsales.csv", header=True, inferSchema=True)
retail_df.head(3)
```

```
[Row(product_id='P0001', store_id='S0002', date='1/2/2017', sales=0.0, revenue=0.0, stock=8, price=6.25, promo_type_1='PR14', promo_bin_1=None, promo_type_2='PR03'),
 Row(product_id='P0001', store_id='S0012', date='1/2/2017', sales=1.0, revenue=5.3, stock=0, price=6.25, promo_type_1='PR14', promo_bin_1=None, promo_type_2='PR03'),
 Row(product_id='P0001', store_id='S0013', date='1/2/2017', sales=2.0, revenue=10.59, stock=0, price=6.25, promo_type_1='PR14', promo_bin_1=None, promo_type_2='PR03')]
```

```python
type(retail_df)
```

```
pyspark.sql.dataframe.DataFrame
```

#### Facing Issue in filtering
- Since I'm not following the code exactly, the notebook actually reads the textfile as a raw file and then processes it. I think that is not needed since it is already a csv file, so let's see what is going wrong here. Btw, here's the original code from the lab.
```python
from pyspark import SparkContext
from datetime import datetime

# Initialize Spark context
sc = SparkContext(appName="RetailStoreSalesAnalysis")

# !wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/XXlNzqYcxqkTbllc-tL_0w/Retailsales.csv

raw_data = sc.textFile("Retailsales.csv")
type(raw_data)
```

```shell
pyspark.rdd.RDD
```

```python
# Parse and Clean Data
def parse_line(line):
    # Split the line by comma to get fields
    fields = line.split(",")
    # Return a dictionary with parsed fields
    return {
        'product_id': fields[0],
        'store_id': fields[1],
        'date': fields[2],
        'sales': float(fields[3]),
        'revenue': float(fields[4]),
        'stock': float(fields[5]),
        'price': float(fields[6]),
        'promo_type_1': fields[7],
        'promo_type_2': fields[9]
    }

# Remove the header line
header = raw_data.first()

raw_data_no_header = raw_data.filter(lambda line: line != header)

# Parse the lines into a structured format
parsed_data = raw_data_no_header.map(parse_line)
parsed_data = parsed_data.filter(lambda x: x is not None)


# Filter out records with missing or invalid data
cleaned_data = parsed_data.filter(lambda x: x['sales'] > 0 and x['price'] > 0)
```

Anywho, here's where I'm stuck
```python
clean_retail_data = retail_df.filter(lambda x: x['sales'] > 0 and x['price'] > 0)
clean_retail_data
```

```shell
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
/tmp/ipykernel_1282/1861704573.py in <module>
----> 1 clean_retail_data = retail_df.filter(lambda x: x['sales'] > 0 and x['price'] > 0)
      2 clean_retail_data

~/spark-2.4.3/python/pyspark/sql/dataframe.py in filter(self, condition)
   1360             jdf = self._jdf.filter(condition._jc)
   1361         else:
-> 1362             raise TypeError("condition should be string or Column")
   1363         return DataFrame(jdf, self.sql_ctx)
   1364 

TypeError: condition should be string or Column
```

Fixed it by explicitly using the `col` instead of `lambda` expressions.
```python
from pyspark.sql.functions import col

clean_retail_data = retail_df.filter((col('sales') > 0) & (col('price') > 0))
clean_retail_data.show(5)
```

```
+----------+--------+--------+-----+-------+-----+-----+------------+-----------+------------+
|product_id|store_id|    date|sales|revenue|stock|price|promo_type_1|promo_bin_1|promo_type_2|
+----------+--------+--------+-----+-------+-----+-----+------------+-----------+------------+
|     P0001|   S0012|1/2/2017|  1.0|    5.3|    0| 6.25|        PR14|       null|        PR03|
|     P0001|   S0013|1/2/2017|  2.0|  10.59|    0| 6.25|        PR14|       null|        PR03|
|     P0001|   S0056|1/2/2017|  1.0|    5.3|    6| 6.25|        PR14|       null|        PR03|
|     P0001|   S0103|1/2/2017|  1.0|    5.3|   10| 6.25|        PR14|       null|        PR03|
|     P0001|   S0106|1/2/2017|  1.0|    5.3|    3| 6.25|        PR14|       null|        PR03|
+----------+--------+--------+-----+-------+-----+-----+------------+-----------+------------+
only showing top 5 rows
```

The lab code next runs the `getNumPartitions()` method but it is a RDD specific method so our code fails, but this can be fixed by converting out DataFrame to RDD.
```python
# Original Code in the Lab
print(f"Number of partitions in cleaned_data: {cleaned_data.getNumPartitions()}")


# My code
num_partitions = clean_retail_data.rdd.getNumPartitions()
print(num_partitions)
```

```
>> Number of partitions in cleaned_data: 2
>>
>> 8
```

#### RDD vs DataFrames
- `RDDs` are the OG way of doing things, whereas `DataFrames` represent the modern way. DataFrame uses the `Catalyst Optimiser`, Spark's internal engine to plan and optimise DataFrame operations.

```python
# Function to count the number of records in each partition
def count_in_partition(index, iterator):
    count = sum(1 for _ in iterator)
    yield (index, count)

# Get the count of records in each partition
partitions_info = cleaned_data.mapPartitionsWithIndex(count_in_partition).collect()
print("Number of records in each partition:")
for partition, count in partitions_info:
    print(f"Partition {partition}: {count} records")

# Get the count of records in each partition
partitions_info = clean_retail_data.rdd.mapPartitionsWithIndex(count_in_partition).collect()
print("Number of records in each partition:")
for partition, count in partitions_info:
    print(f"Partition {partition}: {count} records")
```

```shell
[Stage 8:>                                                          (0 + 2) / 2]

Number of records in each partition:
Partition 0: 97534 records
Partition 1: 99127 records


[Stage 9:=============================>                             (4 + 4) / 8]

Number of records in each partition:
Partition 0: 26115 records
Partition 1: 26821 records
Partition 2: 26137 records
Partition 3: 27749 records
Partition 4: 25618 records
Partition 5: 27776 records
Partition 6: 26866 records
Partition 7: 9579 records
```

#### `mapPartitionsWithIndex`
- `mapPartitionsWithIndex` is a transformation on an `RDD`. Unlike `map`, which processes data one record at a time, `mapPartitionsWithIndex` applies a function to every partition of the RDD. The key part is that it provides two arguments to the function you pass:
	- `index`: The integer index of the partition (e.g., 0, 1, 2, ...). This is useful for identifying which partition you are currently working on.
	- `iterator`: An iterator over all the elements within that partition. You can loop through this iterator to process all the records in the partition.
- `.collect()` is an **action** in Spark that triggers the computation and returns all the data from the RDD to the driver program as a local Python list.

#### What Is a Stage?
A **stage** is a physical execution unit in Spark. It represents a group of computations that can be run together on the same set of data partitions. Stages are created by the **DAG (Directed Acyclic Graph) Scheduler** when a Spark application is run.


#### Aggregations
- Total Sales and Revenue per Product
```python
# Aggregation 1: Total Sales and Revenue per Product
sales_revenue_per_product = cleaned_data.map(lambda x: (x['product_id'], (x['sales'], x['revenue']))).reduceByKey(lambda a, b: (a[0] + b[0], a[1] + b[1]))
print(f"Number of partitions in cleaned_data: {cleaned_data.getNumPartitions()}")

sales_revenue_per_product = clean_retail_data.rdd.map(lambda x: (x['product_id'], (x['sales'], x['revenue']))).reduceByKey(lambda a, b: (a[0] + b[0], a[1] + b[1]))
print(f"Number of partitions in cleaned_data: {clean_retail_data.rdd.getNumPartitions()}")
```

- Total Sales and Revenue per Store
```python
# Aggregation 2: Total Sales and Revenue per Store
sales_revenue_per_store = cleaned_data.map(lambda x: (x['store_id'], (x['sales'], x['revenue']))).reduceByKey(lambda a, b: (a[0] + b[0], a[1] + b[1]))

sales_revenue_per_store = clean_retail_data.rdd.map(lambda x: (x['store_id'], (x['sales'], x['revenue']))).reduceByKey(lambda a, b: (a[0] + b[0], a[1] + b[1]))

```


# Conclusion
Taking a Break here since i feel I have reached my saturation point and also that I need to learn more about the difference between RDDs and DataFrames as well as the difference between respective operations before moving ahead in any useful sense. See you next time where I formalise these notes and move them to [Machine Learning with Apache Spark](Machine%20Learning%20with%20Apache%20Spark.md).