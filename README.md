# Analysing Sales Data Using PySpark

## Introduction
This project is designed to uncover valuable insights into sales trends, customer behaviour, and product performance.In this tutorial, we will explore how to use PySpark, a powerful analytics framework for large-scale data processing, to calculate and analysis the percentage sales yearly and quarterly. We will analysis total sales by country. We’ll go step-by-step through loading the data, preparing it for analysis, and performing the calculations.

## Datasets:
	
The analysis makes use of two primary datasets:
•	Sales Transaction Dataset:
Contains detailed information about each sales transaction.Includes customer ID, product ID, order_date, location and source_order.

•	Product Information Dataset:
Includes product_id, product_name, and product price.
 
```
•	PySpark on google colab and Import libraries needed  for the project
!pip install pyspark
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType,StructField, StringType, IntegerType,DateType
from pyspark.sql.functions import month,year,quarter
from pyspark.sql.functions import count,countDistinct
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

```
•	We start with creating the PySpark session

```
spark = SparkSession \
    .builder \
    .appName("Python Spark SQL basic example") \
    .config("spark.some.config.option", "some-value") \
    .getOrCreate()

```

•	Loading Data and creating schema 
We’ll begin by loading our sales and product data into a PySpark DataFrame.  

```
mySchema = StructType([
    StructField('product_id', IntegerType(), True),
    StructField('customer_id', StringType(), True),
    StructField('order_date', DateType(), True),
    StructField('location', StringType(), True),
    StructField('source_order', StringType(), True)
])

sales_df=spark.read.format("csv").option("inferschema","true").schema(mySchema).load("/content/drive/MyDrive/Colab Notebooks/pysparpro/sales.csv")


menuSchema = StructType([
    StructField('product_id', IntegerType(), True),
    StructField('product_name', StringType(), True),
    StructField('price', StringType(), True)

])

menu_df=spark.read.format("csv").option("inferschema","true").schema(menuSchema).load("/content/drive/MyDrive/Colab Notebooks/pysparpro/menu.csv")
menu_df.show()

```

##  Analysis:
The analysis is structured around key aspects of the business To analyse sales yealy, we’ll add a new columns order_month, order_year and order_quarter to our Data_Frame using the `withColumn()` function from PySpark functions. This function creates a new column the quarter from the order_date column

```
#Deriving year month, quarter
sales_df=sales_df.withColumn("order_month",month(sales_df.order_date))
sales_df=sales_df.withColumn("order_year",year(sales_df.order_date))
sales_df=sales_df.withColumn("order_quarter",quarter(sales_df.order_date))
sales_df.show()

```

•	Merging the two data_frames and counting the amount spent by each customer, the two data _frames are joined on the common column on the two data_frames,’product_id’.PySpark provides powerful aggregation functions for computing summary on columns. It involves summarising data using   groupBy() on  customer_id   and an agg()  function sum on the price column.

```
#Total Amount spent by each customer

total_amount_spent= sales_df.join(menu_df,'product_id').groupBy('customer_id').agg({'price':'sum'}).orderBy('customer_id')
total_amount_spent.rename(columns'{sum(price)' : 'Price'})

total_amount_spent.show()

```

•	Frequency of customer visited 
PySpark function filter() is used to create an new Data Frame by filtering  the elements from an existing Dataframe  based on the condition (sales_df.source_order=='Restaurant')
```
#frequency of customer visted
df_dist=sales_df.filter(sales_df.source_order=='Restaurant').groupBy('customer_id').agg(countDistinct('order_date'))
df_dist.show()
```

```
+-----------+--------------------------+
|customer_id|count(DISTINCT order_date)|
+-----------+--------------------------+
|          E|                         5|
|          B|                         6|
|          D|                         1|
|          C|                         3|
|          A|                         6|
+-----------+--------------------------+
```

•	Total sales by each country 

```
#Total sales by each country
total_amount_country= sales_df.join(menu_df,'product_id').groupBy('location').agg({'price':'sum'})
total_amount_country.show()

```

```
+--------+----------+
|location|sum(price)|
+--------+----------+
|   India|    4860.0|
|     USA|    2460.0|
|      UK|    7020.0|
+--------+----------+

```

•	Total sales by order_source

```
#Total sles by order_source
total_amount_order_source= sales_df.join(menu_df,'product_id').groupBy('source_order').agg({'price':'sum'})
total_amount_order_source.show()

```

```
+------------+----------+
|source_order|sum(price)|
+------------+----------+
|      zomato|    4920.0|
|      Swiggy|    6330.0|
|  Restaurant|    3090.0|
+------------+---------

```

## Dashboard
Data visualization is a powerful tool for exploring and analysing data. PySpark provides integration with popular visualization libraries, such as Matplotlib. The data_ frame was converted to pandas.



  



 
