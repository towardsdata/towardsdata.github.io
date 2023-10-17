---
layout: post
title:  "Lakehouse's data management strategies"
kicker: "Lakehouse's data management strategies"
subtitle: "The modern data lake is referred to as a 'lakehouse', which is a hybrid architecture that combines the best of a data lake and a data warehouse to address the shortcomings and challenges of data management associated with the data lake."
image: assets/images/posts-cover-images/lakehouse-data-management.png
imageshadow: true
toc: true
author: senthil
date: 2023-10-17 00:000:00 +0530
tags: [ "storage-strategies", "copy-on-write", "merge-on-read", "cow", "mor", "lakehouse" ]
categories: lakehouse
featured: false
hidden: false
---

# Overview

Let's go over in depth in this post how modern data lakes perform **deletes** and **upserts**, much like OLTP databases. An "upsert" is a mix of "update" and "insert," and it refers to the process modifying an existing record in a data store if it already exists or adding a new record if it doesn't. Updates and deletions are fundamental operations in any OLTP database. These OLTP databases use row-level storage with built-in indexes, which makes it faster to locate a particular record for upserts. 

> **Indices vs. indexes**: Both "indexes" and "indices" are acceptable plural forms of the word "index".

There is no built-in support for upserts in a traditional Hadoop, aka data lake, which comprises the Hadoop Distributed File System (HDFS) and a number of data processing frameworks including Apache MapReduce, Apache Spark, and Apache Hive. Data lake's primary use case was to process and analyze a huge volume of data in batch mode. Since real-time deletes and upserts are not naturally suited to this form of operation, ACID (Atomicity, Consistency, Isolation, Durability) transactions were not intended to be supported by it, and it wasn't part of the design objective.

In traditional data lakes, the Apache Spark approach reads and then overwrites the whole table or partition with each update to handle batch updates, even for small changes. We are all well aware how bad it was.

Modern data lakes have emerged to solve these transaction abilities, such as updates and deletes. The modern data lake is known as a **lakehouse**, which has chosen a hybrid architecture that combines the best aspects of a data lake and a data warehouse, which were historically designed with different trade-offs in mind. 

There are three popular lakehouses at the present time: **Apache Hudi**, **Apache Iceberg**, and **Delta Lake**. The primary features of the data warehouse used in lakehouse are indexing and ACID transactions. Indexing and ACID transactions are critical for today’s lakehouse. For instance, some important use cases, such as data retention and change data capture (CDC), rely heavily on ACID transactions.

# Apache Parquet is the primary and preferred storage file format for lakehouse

Lakehouse has chosen **Apache Parquet** as its preferred and primary storage file format. For a number of reasons, Apache Hudi, Apache Iceberg, and Delta Lake built their lakehouses on top of this Apache Parquet file format in their efforts to offer stability, performance, and ACID transaction support on top of data lakes. The following are the key reasons to go with the Parquet file format:

* **Columnar storage**: Parquet is a *columnar storage* file format, which means that data is stored column by column rather than row by row. This columnar storage layout is highly efficient for analytical queries because it allows for predicate pushdown and compression, resulting in reduced I/O and improved query performance.
* **Efficient compression**: Parquet supports various compression algorithms, which can significantly reduce the storage footprint and improve query performance. Columnar storage further enhances compression by grouping similar data together.
* **Schema evolution**: Schema evolution is a feature that enables us easily change a table's current schema to accommodate data that is evolving over time without affecting existing data.
* **Column pruning**: Columnar storage can be a good choice when queries do not read all columns of the data. This is known as column pruning and may result in a significant speed boost.

# Lakehouse's data management strategies

Let's go over some key data management strategies for a lakehouse. Apache Hudi, Delta Lake, and Apache Iceberg are three popular open-source projects that aim to improve the management and processing of data in data lakes. 

## Update and delete operations

When dealing with "updates" and "deletes" in traditional data lakes, the entire dataset or specific partition(s) must be rewritten. It is a tedious and time-consuming activity. It's crucial for today's lakehouse to support update and delete operations with ACID properties. Each lakehouse has its own approach to handling data updates or deletes, and differing design choices relate to the concepts of **Copy-on-Write** (**CoW**) and **Merge-on-Read** (**MoR**). Typically, these Lakehouse architectures often combine CoW and MoR strategies, depending on the specific needs and design principles of the lakehouse architecture. For instance, in Apache Hudi, Copy-on-Write (CoW) is the default storage technique, whereas Merge-on-Read (MoR) is an optional one, letting users to choose the strategy that best fits their use cases. So in the case of Apache Hudi, both Copy-on-Write and Merge-on-Read coexist.

What exactly are Copy-on-Write and Merge-on-Read? Note that these are not only the two concepts of data management strategies but also two different dataset types. That's right, you got it :-) Infact, the specifics of how the data is laid out as files depend on one of these dataset types that we choose. Understanding each of these strategies—Copy-on-Write as well as Merge-on-Read—in depth will help us go forward.

Let's see how one of the Lakehouse frameworks uses the Cow and MoR strategies to deal with updates and deletes. We'll make use of the Apache Hudi framework to walk through these strategies, but a similar idea can apply to Delta Lake and Apache Iceberg.

### Copy-on-Write (CoW)

Copy-on-Write (CoW) is a sort of optimized variant of the older method of performing delete and update operations. Having said that, Copy-on-Write is the default storage technique in Apache Hudi and it stores data in a columnar format (Parquet). With Copy-on-Write datasets, each time there is an update to a record, the file that contains the record is rewritten into a *new version of the file* with the updated values. Updates may affect more than one underlying data file, so the Copy-on-Write (CoW) mechanism typically creates multiple new delta files during a write operation rather than a single delta file. Note that more updates result in more versioned Parquet files.

The underlying table data is physically stored as individual files, which are often grouped into *partitions* (directories) using date-time or other partitioning methods. At a high level, this is how updates work: Apache Hudi locates the impacted files in each partition using one of the indexing techniques, then reads them completely, updates them with new values in memory, and finally writes to disk and creates new files.

Let's try some code to test how the default behavior (Copy-on-Write) works when we try to modify the existing records in a Hudi table. We will run our exercises in PySpark with Hudi. The Apache Spark version used here is 3.4. 

```bash
export PYSPARK_PYTHON=$(which python3)
export SPARK_VERSION=3.4

pyspark --packages org.apache.hudi:hudi-spark$SPARK_VERSION-bundle_2.12:0.14.0 \
        --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
        --conf 'spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog' \
        --conf 'spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension' \
        --conf 'spark.kryo.registrator=org.apache.spark.HoodieSparkKryoRegistrar'
```

First, let's create an employee table with some sample data. In the code example that follows, we create a Spark DataFrame with some sample data, and then we write that DataFrame into the Hudi table. Hudi builds a partitioned table here since we use the `department` column as a partitioned column. 

> **Overwrite mode**: We used "overwrite" mode, which overwrites and recreates the table if it already exists. In other words, it tells Spark to delete the table and recreate it.

```bash
>>> tableName = "employees_table"
>>> basePath = "file:///tmp/employees_table"
>>> columns = ["id","name","age","salary","department"]
>>> data =[(1695159649087,"John Sundar",43,2000,"data engineering"),
...        (1695091554788,"Senthil Nayagan",40,2300,"data science"),
...        (1695046462179,"Arun Anand",37,1800,"business analyst"),
...        (1695516137016,"Jeff Parks",50,5000,"business analyst"),
...        (1695115999911,"Vicky",39,2400,"data engineering")]
>>> inserts = spark.createDataFrame(data).toDF(*columns)
>>> inserts.show()
+-------------+---------------+---+------+----------------+
|           id|           name|age|salary|      department|
+-------------+---------------+---+------+----------------+
|1695159649087|    John Sundar| 43|  2000|data engineering|
|1695091554788|Senthil Nayagan| 40|  2300|    data science|
|1695046462179|     Arun Anand| 37|  1800|business analyst|
|1695516137016|     Jeff Parks| 50|  5000|business analyst|
|1695115999911|          Vicky| 39|  2400|data engineering|
+-------------+---------------+---+------+----------------+

>>> hudi_options = {
...     'hoodie.table.name': tableName,
...     'hoodie.datasource.write.partitionpath.field': 'department'
... }
>>> inserts.write.format("hudi"). \
...     options(**hudi_options). \
...     mode("overwrite"). \
...     save(basePath)
```

As we can see below, Hudi created a partitioned table called `employees_table` under the `/tmp` directory and Parquet files are created in the respective partitioned directory.

```bash
sasen@Sasens-MacBook-Pro employees_table % pwd
/tmp/employees_table
sasen@Sasens-MacBook-Pro employees_table % tree
.
├── business analyst
│   ├── 1cc0cda4-1299-4535-a493-b33666c67dc6-0_7-15-0_20231013171231634.parquet
│   └── c01dd0fc-fb33-4f25-8594-00f29dc599ec-0_9-17-0_20231013171231634.parquet
├── data engineering
│   ├── dca9ea05-c600-4f49-a45b-a9e21f34434d-0_2-10-0_20231013171231634.parquet
│   └── f984d73e-ebd1-4393-8005-cbeb530a6f2c-0_11-19-0_20231013171231634.parquet
└── data science
    └── 1fc7ec73-7f15-44a4-83df-e4040d87512c-0_4-12-0_20231013171231634.parquet

3 directories, 5 files
```

Let's do an update and see how CoW works. We will update the age of `Senthil Nayagan` to `42`.

```bash
from pyspark.sql.functions import lit, col

updatesDf = spark.read.format("hudi").load(basePath).filter("name == 'Senthil Nayagan'").withColumn("age", col("age")+2)

updatesDf.write.format("hudi"). \
  options(**hudi_options). \
  mode("append"). \
  save(basePath)
```

During the update, Hudi uses one of the indexing methods to locate the impacted files in each partition, then reads them completely, updates the `age` field in memory, and finally creates a new file. The highlighted file in the below screenshot shows a new file that was rewritten with an updated age value of `42`.

> **Append mode**: Note that updates use `append` mode. When `append` mode is used, Spark appends the new data to existing old data on disk or cloud storage. Hudi recommends using `append` mode always, unless we are trying to create the table for the first time.

|![Figure 1: Hudi update created a new delta file](/assets/images/posts/hudi-update.png){: width="100%" }|
|:-:|
|<sup>*Figure 1: Hudi update created a new delta file.*</sup>|<br/><br/>

#### Updates affecting more than one underlying data file 

Updates may affect more than one underlying data file, depending on the update criteria used. Consider a scenario where we update the salary if the employee is older than `40`. The updates in this case affect three records that are spread across various partitions.

```bash
>>> updatedSalaryDF = employeesDF.filter("age > 40").withColumn("salary", col("salary")+200)
>>> updatedSalaryDF.show()
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+---------------+---+------+----------------+
|_hoodie_commit_time|_hoodie_commit_seqno|  _hoodie_record_key|_hoodie_partition_path|   _hoodie_file_name|           id|           name|age|salary|      department|
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+---------------+---+------+----------------+
|  20231013183529453|20231013183529453...|20231013171231634...|          data science|1fc7ec73-7f15-44a...|1695091554788|Senthil Nayagan| 42|  2500|    data science|
|  20231013171231634|20231013171231634...|20231013171231634...|      data engineering|dca9ea05-c600-4f4...|1695159649087|    John Sundar| 43|  2200|data engineering|
|  20231013171231634|20231013171231634...|20231013171231634...|      business analyst|c01dd0fc-fb33-4f2...|1695516137016|     Jeff Parks| 50|  5200|business analyst|
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+---------------+---+------+----------------+
```

In this case, the Copy-on-Write created multiple delta files, highlighted below:

|![Figure 2: Hudi update affected multiple files](/assets/images/posts/hudi-update-effects-multi-files.png){: width="100%" }|
|:-:|
|<sup>*Figure 2: Hudi update affected multiple files.*</sup>|<br/><br/>

Having that, with the Copy-on-Write table, each time a record is modified, the file containing the record is rewritten into a new version of the file with the updated values as shown below:

|![Figure 3: Copy-on-Write - How it works?](/assets/images/posts/hudi-cow-how-it-works.png){: width="100%" }|
|:-:|
|<sup>*Figure 3: Copy-on-Write - How it works?*</sup>|<br/><br/>

#### Updates made to a single Hudi table with many rows of data

Consider a scenario in which an update is performed on a single Hudi table with multiple rows of data. Let's use the same employee data. This time, the data will not be partitioned in order to generate a single Hudi Parquet table.

```bash
tableName = "emp_single_hudi"
basePath = "file:///tmp/emp_single_hudi"

columns = ["id","name","age","salary","department"]
data =[(1695159649087,"John Sundar",43,2000,"data engineering"),
       (1695091554788,"Senthil Nayagan",40,2300,"data science"),
       (1695046462179,"Arun Anand",37,1800,"business analyst"),
       (1695516137016,"Jeff Parks",50,5000,"business analyst"),
       (1695115999911,"Vicky",39,2400,"data engineering")]

inserts = spark.createDataFrame(data).toDF(*columns)

hudi_options = {
    "hoodie.table.name": tableName, 
    "hoodie.datasource.write.operation": "insert",
}

df.write \
    .format("hudi") \
    .options(**hudi_options) \
    .mode("overwrite") \
    .save(basePath)
```

With the above code snippet, we were able to create a single Hudi Parquet, as shown below:

```bash
sasen@Sasens-MacBook-Pro emp_single_hudi % tree -D -t
[Oct 14 17:50]  .
└── [Oct 14 17:50]  435187f7-ac3c-41e8-b149-0a3fb496548e-0_0-15-14_20231014174958919.parquet

0 directories, 1 file
```

Let's update a specific row. For example, update `Vicky`'s age to `42`. Following the update, check the underlying Parquet file.

```bash
>>> from pyspark.sql.functions import lit, col
>>> updatesDf = spark.read.format("hudi").load(basePath).filter("name == 'Vicky'").withColumn("age", col("age")+3)
>>> updatesDf.filter("name == 'Vicky'").show()
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+-----+---+------+----------------+
|_hoodie_commit_time|_hoodie_commit_seqno|  _hoodie_record_key|_hoodie_partition_path|   _hoodie_file_name|           id| name|age|salary|      department|
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+-----+---+------+----------------+
|  20231014174958919|20231014174958919...|20231014174958919...|                      |435187f7-ac3c-41e...|1695115999911|Vicky| 42|  2400|data engineering|
+-------------------+--------------------+--------------------+----------------------+--------------------+-------------+-----+---+------+----------------+

>>> updatesDf.write.format("hudi"). \
...   options(**hudi_options). \
...   mode("append"). \
...   save(basePath)
```

Now we reach an important point that needs attention. After the update, Hudi's Copy-on-Write method created an additional Parquet file (created at 17:57), as shown below:

```bash
sasen@Sasens-MacBook-Pro emp_single_hudi % tree -D -t
[Oct 14 17:57]  .
├── [Oct 14 17:50]  435187f7-ac3c-41e8-b149-0a3fb496548e-0_0-15-14_20231014174958919.parquet
└── [Oct 14 17:57]  435187f7-ac3c-41e8-b149-0a3fb496548e-0_0-36-35_20231014175745613.parquet

0 directories, 2 files
```

As we see, the original file that was created at 17:50 remains unchanged during the update. Under the hood, Hudi's CoW created a new version of the original file and appended the modified row to the new file.

```bash
>>> existingFile = spark.read.parquet('./emp_single_hudi/435187f7-ac3c-41e8-b149-0a3fb496548e-0_0-15-14_20231014174958919.parquet')
>>> newFile = spark.read.parquet('./emp_single_hudi/435187f7-ac3c-41e8-b149-0a3fb496548e-0_0-36-35_20231014175745613.parquet')
>>> existingFile.select("id", "name", "age", "salary", "department").show()
+-------------+---------------+---+------+----------------+
|           id|           name|age|salary|      department|
+-------------+---------------+---+------+----------------+
|1695159649087|    John Sundar| 43|  2000|data engineering|
|1695091554788|Senthil Nayagan| 40|  2300|    data science|
|1695046462179|     Arun Anand| 37|  1800|business analyst|
|1695516137016|     Jeff Parks| 50|  5000|business analyst|
|1695115999911|          Vicky| 39|  2400|data engineering|
+-------------+---------------+---+------+----------------+

>>> newFile.select("id", "name", "age", "salary", "department").show()
+-------------+---------------+---+------+----------------+
|           id|           name|age|salary|      department|
+-------------+---------------+---+------+----------------+
|1695159649087|    John Sundar| 43|  2000|data engineering|
|1695091554788|Senthil Nayagan| 40|  2300|    data science|
|1695046462179|     Arun Anand| 37|  1800|business analyst|
|1695516137016|     Jeff Parks| 50|  5000|business analyst|
|1695115999911|          Vicky| 39|  2400|data engineering|
|1695115999911|          Vicky| 42|  2400|data engineering|
+-------------+---------------+---+------+----------------+
```

#### Advantages of CoW

- **Immutability**: One of the primary advantages of CoW is immutability. In CoW, once a data record is written, it is never modified. Instead, a new version of the record is created every time an update occurs. This immutability is beneficial for maintaining data lineage
- **Optimized for query performance**: CoW can be optimized for read-heavy workloads because the data remains immutable (unchanged) and multiple read operations can be performed simultaneously without any locking or blocking issues. Thus, it is an ideal choice for scenarios where there are a high number of read operations.

#### Disadvantages of CoW

- Write cost and write latency is higher.
- 

#### Use cases

- Great for fast query or read performance.
- To store append-only data (never updated) like event logs.
  
Since pictures speak louder than words, we've illustrated below how Copy-on-Write works through a sequence of commits:

|![Figure 4: Copy-on-Write - How it works with a series of commits](/assets/images/posts/hudi-cow-how-it-works-illustration.png){: width="100%" }|
|:-:|
|<sup>*Figure 4: Copy-on-Write - How it works with a series of commits.*</sup>|<br/><br/>

### Merge-on-Read (MoR)

Merge-on-Read was pioneered by Apache Hudi for situations where Copy-on-Write may not be optimal. Despite the fact that CoW provides substantial cost and time savings, we must still rewrite a portion of the data. In the case of streaming data with CoW, more table updates resulted in more file versions and an increase in file count. This led to inefficient writes and less fresh data.

Merge-on-Read (MoR) was the other storage table type created for Hudi to reduce the write amplification with frequent updates. Merge-on-Write table is a little bit more complicated than a Copy-on-Write table for a couple of reasons. During an update, rather than rewriting the entire files with updated values into the new version of files, MoR, on the other hand, writes updates to separate row-based delta log files. In other words, when we initially create an ingestion on MoR tables, it is written into columnar Parquet files, but any subsequent ingestions are stored in the delta log files. Hudi uses **Apache Avro** to store these delta logs.

> **Note:** It's worth noting that with MoR, the entire files or partitions are not rewritten; instead, only the changes are written to new log files. This makes writing the changes much quicker, but reading the data requires additional effort.

These delta logs are compacted into new columnar-based files either synchronously or asynchronously, and a new version of the columnar-based base file (Parquet) is generated in order to improve query performance. Frequent commits are possible with the MoR table type, but may not be possible with the CoW table type.

The following illustrates how Merge-on-Read works with a sequence of updates:

|![Figure 5: Merge-on-Read - How it works with a series of commits](/assets/images/posts/hudi-mor-how-it-works-illustration.png){: width="100%" }|
|:-:|
|<sup>*Figure 5: Merge-on-Read - How it works with a series of commits.*</sup>|<br/><br/>

#### Use cases

- Best for update-heavy (frequent updates) tables where we want faster and efficient writes.
- 

### Copy-on-Write vs. Merge-on-Read

todo

# Frequently asked questions (FAQ)

## How can I know what kind (Copy-on-Write or Merge-on-Write) of Hudi table this is?

To determine whether a Hudi table is using Copy-on-Write or Merge-on-Write storage, one can inspect the metadata and configuration associated with the Hudi table. One way to look at it is to check the `hoodie.properties` file, which can be found in our Hudi table directory. If our Hudi table is `employees_table`, then a hidden sub-directory named `.hoodie` can be found inside. Check the `hoodie.properties` file in the `.hoodie` subdirectory to see the type of table.

```bash
sasen@Sasens-MacBook-Pro .hoodie % cat hoodie.properties
#Updated at 2023-10-13T11:42:37.977856Z
#Fri Oct 13 17:12:37 IST 2023
hoodie.table.type=COPY_ON_WRITE
```

## What is "record key" in Apache Hudi?

The "record key" is a fundamental concept in Apache Hudi used to uniquely identify records or rows within a Hudi dataset. It plays a crucial role in the organization and management of data, particularly when dealing with upserts and merge operations.

Here's a more detailed explanation:

- **Uniqueness**: The record key is used to ensure that each record in a Hudi table is unique. Hudi requires that each record has a unique record key in order to maintain data integrity.
- **Organization of data**: The record key is used to organize data within Hudi tables. Data records with the same record key are logically grouped together, often referred to as a "partition." This organization enables efficient data retrieval and manipulation.
- **Upserts**: Hudi supports operations like upserts, which involve inserting new records and updating existing records based on their record keys. The record key is crucial for determining which records should be updated and which ones should be inserted as new records.
- **Indexing**: The record key is frequently used to create an index that enables fast lookups and retrieval of specific Hudi dataset records. This is particularly essential for point queries, which retrieve a specific record based on its record key.
- **Consistency and ACID transactions**: The record key is used to enforce data consistency through ACID transactions by preventing duplicate or conflicting records.
- **Customized record key**: We can choose a field within our data as the record key or even generate a synthetic key if needed. The choice of the record key depends on our data and use case.

## How can we write data into the Hudi dataset/table?

We can write data to the Hudi dataset using the Spark Data Source API or the Hudi DeltaStreamer utility.

## What's the property in Hudi that defines the table/dataset type (either Copy-on-Write or Merge-on-Read)?

We can use the `hoodie.datasource.write.table.type` property to set the table/dataset type. The default value is `COPY_ON_WRITE`. For a MoR table, set this value to `MERGE_ON_READ`.