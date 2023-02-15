# Apache Spark Big Data Analytics
> Our team has prepared a set of data containing search terms on our e-Commerce platform. Download the data and run analytic queries on it using `pyspark` and `JupyterLab`. Use a pretrained sales forecasting model to predict the sales for 2023.

## 1. Install `pyspark`

```python
!pip install pyspark
!pip install findspark
```

## 2. Creating the Spark Session and Context

```python
import findspark
findspark.init()

from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession

# Creating a spark context class
sc = SparkContext()

# Creating a spark session
spark = SparkSession \
    .builder \
    .appName("Saving and Loading a SparkML Model").getOrCreate()
```

## 3. Analyzing Search Terms

First I will download the source file that contains our search term data `searchterms.csv` and load it into a dataframe `seach_term_dataset`.
```python
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/Bigdata%20and%20Spark/searchterms.csv
src = 'searchterms.csv'
search_term_dataset = spark.read.csv(src)
```

I can inspect the dataframe by displaying the number of rows and columns.
```python
row_count = search_term_dataset.count()
col_count = len(search_term_dataset.columns)
print(f'Row count for search_term_dataset: {row_count}')
print(f'Column count for search_term_dataset: {col_count}')
```
> ```
> Row count for search_term_dataset: 10001
> Column count for search_term_dataset: 4
> ```

Let's view the first 5 rows of our dataset to get an overview of the data we're working with. There are 4 columns `_c0`, `_c1`, `_c2`, and `_c3` attributed to search `day`, `month`, `year`, and `searchterm` respectively.
```python
search_term_dataset.show(5)
```
> ```
> +---+-----+----+--------------+
> |_c0|  _c1| _c2|           _c3|
> +---+-----+----+--------------+
> |day|month|year|    searchterm|
> | 12|   11|2021| mobile 6 inch|
> | 12|   11|2021| mobile latest|
> | 12|   11|2021|   tablet wifi|
> | 12|   11|2021|laptop 14 inch|
> +---+-----+----+--------------+
> only showing top 5 rows
> ```

Now I will display the datatype for the `searchterm` column.
```python
print(search_term_dataset.schema['_c3'].dataType)
```
> ```
> StringType
> ```

Suppose we wanted to know how many times the term `gaming laptop` was searched. I can determine this with the following code:
- Create a Spark view from the dataframe
- Run a simple COUNT() query with `spark.sql` where `_c3` is equal to `gaming laptop`
```python
search_term_dataset.createOrReplaceTempView('searches')
spark.sql('SELECT COUNT(*) FROM searches WHERE _c3="gaming laptop"').show()
```
> ```
> +--------+
> |count(1)|
> +--------+
> |     499|
> +--------+
> ```

What if we wanted to see the top 5 most frequently used search terms? Using `spark.sql` I can select the search terms and group them by their count, with a descending order by count.
```python
spark.sql('SELECT _c3, COUNT(_c3) FROM searches GROUP BY _c3 ORDER BY COUNT(_c3) DESC').show(5)
```
> ```
> +-------------+----------+
> |          _c3|count(_c3)|
> +-------------+----------+
> |mobile 6 inch|      2312|
> |    mobile 5g|      2301|
> |mobile latest|      1327|
> |       laptop|       935|
> |  tablet wifi|       896|
> +-------------+----------+
> only showing top 5 rows
> ```

## 4. Sales Forecasting Model

In this portion of the assignment I will be using a pretrained sales forecasting model.
```python
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/Bigdata%20and%20Spark/model.tar.gz
!tar -xvzf model.tar.gz
```

After downloading the model, I can create a dataframe `sales_prediction_df` from the parquet and print the schema.
```python
sales_prediction_parquet = 'sales_prediction.model/data/part-00000-1db9fe2f-4d93-4b1f-966b-3b09e72d664e-c000.snappy.parquet'
sales_prediction_df = spark.read.parquet(sales_prediction_parquet)
sales_prediction_df.printSchema()
```
> ```
> root
>  |-- intercept: double (nullable = true)
>  |-- coefficients: vector (nullable = true)
>  |-- scale: double (nullable = true)
> ```

Now I will load the model into `sales_prediction_model`
```python
from pyspark.ml.regression import LinearRegressionModel
sales_prediction_model = LinearRegressionModel.load('sales_prediction.model')
```

Now I will use the sales forecast model to predict the sales for the year of 2023.
```python
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.regression import LinearRegression
def predict(year):
    assembler = VectorAssembler(inputCols=["year"],outputCol="features")
    data = [[year,0]]
    columns = ["year", "sales"]
    _ = spark.createDataFrame(data, columns)
    __ = assembler.transform(_).select('features','sales')
    predictions = sales_prediction_model.transform(__)
    predictions.select('prediction').show()
predict(2023)
```
> ```
> +------------------+
> |        prediction|
> +------------------+
> |175.16564294006457|
> +------------------+
> ```



## About This Lab

##### Environment/IDE
To complete this lab, we will be using a Skills Network Labs (SN Labs) virtual lab environment.

##### Tools/Software
- JupyterLab
- PySpark

[<kbd> <br> ← Previous Assignment <br> </kbd>](/06%20-%20Apache%20Airflow%20ETL%20&%20Data%20Pipelines)
