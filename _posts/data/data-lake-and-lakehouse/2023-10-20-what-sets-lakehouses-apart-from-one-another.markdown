---
layout: post
title:  "What sets Lakehouses apart from one another?"
kicker: "lakehouse"
subtitle: 'The modern data lake is referred to as a "lakehouse", which is a hybrid architecture that combines the best of a data lake and a data warehouse to address the shortcomings and challenges of data management associated with the traditional data lake. There are three popular lakehouses at the present time: Apache Hudi, Apache Iceberg, and Delta Lake. Although all of these lakehouses have the same objective of overcoming the pain points of data lakes, their unique features and distinct ways of doing things set them apart from one another.'
image: assets/images/posts-cover-images/lakehouse-data-management.png
imageshadow: true
toc: true
author: senthil
date: 2023-10-20 00:000:00 +0530
tags: [ "lakehouses", "transactionsl-data-lake", "delta-lake", "lakehouse" ]
categories: lakehouse
featured: false
hidden: false
draft: true
---

# Overview

There are a number of good lakehouse comparison studies and benchmarking blog posts that cover all aspects of features, but I'd like to focus on a few of the most essential features with statistics. I recommend reading all of those posts to gain a complete understanding of the lakehouse’s features.

The modern data lake is referred to as a "lakehouse", which is a hybrid architecture that combines the best of a data lake and a data warehouse to address the shortcomings and challenges of data management associated with the data lake. There are three popular open-source lakehouses at the present time: **Apache Hudi** (by Uber), **Apache Iceberg** (by Netflix), and **Delta Lake** (by Databricks). Although all of these lakehouses have the same objective of overcoming the pain points of data lakes, their unique features and distinct ways of doing things set them apart from one another.

# A brief introduction to lakehouse and why it emerged as a modern data lake

A lakehouse is a modern distributed data storage and processing architecture that combines the flexibility of a data lake with the structure, schema enforcement, and transactional capabilities of a data warehouse. Simply put, it's the best of both worlds—a data lake and a data warehouse. It emerged as a response to some of the limitations and challenges associated with traditional data lakes.

A data lake is a scalable storage and analysis system for data of any type, but it has some pain points that the data architects find harder to get around with. Here are some common pain points of data lakes:

- **Lack of transactional support with ACID properties**: Data lakes typically lack transactional support (due to lack of transactional log), making it difficult to manage data updates and deletes with the required ACID properties for certain use cases, particularly those requiring GDPR and CCPA compliance.
- **Lack of schema enforcement**: Data lakes do not enforce schema on write, which means data is ingested in its raw form without enforced structure. This can lead to data quality issues, inconsistencies, and challenges when querying and analyzing the data at a later point.
- **Data reliability**: Without ACID properties, data lakes can suffer from reliability issues that make it difficult for data engineers to invest time and effort in removing corrupted data, validating the rest of the data, and establishing a new write task to fill in data gaps.
- **Query performance**: The query performance of data lake analytics tools is a significant factor for user satisfaction. Data lakes can hold a very large volume of data, so it’s crucial that our data lake query engine is optimized for performance at scale. Some of the most common performance issues with data lakes are:
  - **Small files problems**: Having a large number of small files in a data lake can slow down performance considerably due to limitations with I/O throughput.
  - **Lack of indexing**: Data lakes typically do not support traditional indexing in the same way that relational databases do.
  - **Metadata management**: Data lakes that grow to petabytes or more may become bottlenecked not by the data itself but by the metadata that is associated with it.

Modern data lakes have emerged in response to the aforementioned limitations and challenges. The modern data lake is known as a **lakehouse**, which has chosen a hybrid architecture that combines the best aspects of a data lake and a data warehouse, which were historically designed with different trade-offs in mind. Due to its transactional capabilities, the modern data lake is also referred to as a **transactional data lake**. The term "lakehouse" was popularized by Databricks, the company behind Apache Spark, with their open-source project called Delta Lake. There are three popular lakehouses at the present time: **Apache Hudi**, **Apache Iceberg**, and **Delta Lake**. Let's go over the key features of these lakehouses to find out which one is preferred over the others for specific use cases.

## Apache Parquet is the primary and preferred storage file format for lakehouses

Lakehouses have chosen **Apache Parquet** as their preferred and primary storage file format. For a number of reasons, Apache Hudi, Apache Iceberg, and Delta Lake built their lakehouses on top of this Apache Parquet file format in their efforts to offer stability, performance, and ACID transaction support on top of data lakes. The following are the key reasons to go with the Parquet file format:

* **Columnar storage**: Parquet is a *columnar storage* file format, which means that data is stored column by column rather than row by row. This columnar storage layout is highly efficient for analytical queries because it allows for predicate pushdown and compression, resulting in reduced I/O and improved query performance.
* **Efficient compression**: Parquet supports various compression algorithms, which can significantly reduce the storage footprint and improve query performance. Columnar storage further enhances compression by grouping similar data together.
* **Schema evolution**: Schema evolution is a feature that enables us easily change a table's current schema to accommodate data that is evolving over time without affecting existing data.
* **Column pruning**: Columnar storage can be a good choice when queries do not read all columns of the data. This is known as column pruning and may result in a significant speed boost.

# What sets lakehouses apart from one another?

Due to the increasing popularity of the lakehouse, there is a growing need and interest in comparing these open-source lakehouse projects. There are a number of good lakehouse comparison studies and benchmarking blog posts that cover all aspects of features, but in this post, I cover the most essential features and provide statistics.

It is crucial to keep in mind that no single lakehouse is invariably superior in every way and may possibly take over the others. Each lakehouse has its own strengths and weaknesses due to the fact that it evolved over different time periods and had varying goals while addressing the primary pain points of the data lake. The choice between these lake residences must be based on our specific use cases and requirements.

Note that not only does each lakehouse have unique features, but it also does things uniquely. The unique features and unique way of doing things set Lakehouses apart from one another. In the current data landscape, lakehouses are emerging rapidly, and each developer community is working hard to fill in the gaps and introduce new capabilities down the line. Therefore, the one (lakehouse) who lacks specific features today could thrive with the addition of new abilities and take the lead tomorrow. Having said that, we must keep a close watch on the trends that influence these lake residences.

Let's evaluate the performance of each lakehouse on each of the key features.

## Update and delete transactions

Similar to any OLTP datastore, "updates" and "deletes" with ACID properties play a crucial role in maintaining data quality and supporting evolving data needs in today's lakehouse. When dealing with updates and deletes in traditional data lakes, the entire dataset or specific partition(s) must be rewritten. It is a tedious and time-intensive operation. However, lakehouse takes on these update and delete operations in a similar way to a transactional database by using transaction logs.

Each lakehouse has its own approach to handling data updates or deletes, and differing design choices relate to the concepts of **Copy-on-Write** (**CoW**) and **Merge-on-Read** (**MoR**). Typically, these lakehouse architectures often combine CoW and MoR strategies, depending on the specific needs and design principles of the lakehouse architecture. For instance, in Apache Hudi, Copy-on-Write (CoW) is the default storage technique, whereas Merge-on-Read (MoR) is an optional one, letting users to choose the strategy that best fits their use cases. So in the case of Apache Hudi, both Copy-on-Write and Merge-on-Read coexist.

What exactly are Copy-on-Write and Merge-on-Read? Note that these are not only the two concepts of data management strategies but also two different dataset types. That's right, you got it :-) Infact, the specifics of how the data is laid out as files depend on one of these dataset types that we choose. Understanding each of these strategies—Copy-on-Write as well as Merge-on-Read—in depth will help us go forward.

Let's check out how each lakehouse architecture handles updates and deletes using one or both of the Cow and MoR strategies.

### Copy-on-Write (CoW)

Copy-on-Write (CoW) is a sort of optimized variant of the older method of performing delete and update operations in traditional data lakes. With Copy-on-Write datasets, each time there is an update to a record, the file that contains the record is rewritten into a *new version of the file* (new snapshot) with the updated values. The original data files remain immutable (unchanged). This ensures that the original data is preserved and can be accessed consistently, for example, through a time-travel query.

|![Figure 1: Update on CoW table](/assets/images/posts/updates-on-cow-table.png){: width="100%" }|
|:-:|
|<sup>*Figure 1: Update on CoW table.*</sup>|<br/><br/>

Updates may affect more than one underlying data file, so the Copy-on-Write (CoW) mechanism typically creates multiple new delta files during a write operatione. Note that more updates result in more versioned Parquet files.

The underlying table data is physically stored as individual files, which are often grouped into *partitions* (directories) using date-time or other partitioning methods. At a high level, this is how updates work on CoW data files: Apache Hudi locates the impacted files in each partition using one of the indexing techniques, then reads them completely, updates them with new values in memory, and finally writes to disk and creates new files.

Since pictures speak louder than words, I’ve illustrated below how Copy-on-Write works through a sequence of commits:

|![Figure 2: Copy-on-Write - How it works with a series of commits](/assets/images/posts/lakehouse-cow-how-it-works-illustration.png){: width="100%" }|
|:-:|
|<sup>*Figure 2: Copy-on-Write - How it works with a series of commits.*</sup>|<br/><br/>

#### Advantages and disadvantages of CoW

##### Advantages of CoW

- **Immutability**: One of the primary advantages of CoW is immutability. In CoW, once a data record is written, it is never modified. Instead, a new version of the record is created every time an update occurs. This immutability is beneficial for maintaining data lineage
- **Optimized for query performance**: CoW can be optimized for read-heavy workloads because the data remains immutable (unchanged) and multiple read operations can be performed simultaneously without any locking or blocking issues. Thus, it is an ideal choice for scenarios where there are a high number of read operations.

##### Disadvantages of CoW

- Write cost and write latency are higher. 
- Higher write amplification in tandem with higher write latency - Write amplification is very high because it rewrites the whole dataset or partition. Increased write amplification brings higher lantency.


### Merge-on-Read (MoR)

Merge-on-Read was pioneered by Apache Hudi for situations where Copy-on-Write may not be optimal. Despite the fact that CoW provides substantial cost and time savings, we must still rewrite a portion of the data. In the case of streaming data with CoW, more table updates resulted in more file versions and an increase in file count. This led to inefficient writes and less fresh data.

Merge-on-Read (MoR) was the other storage table type created to reduce the write amplification with frequent updates. Merge-on-Write table is a little bit more complicated than a Copy-on-Write table for a couple of reasons. During an update, rather than rewriting the entire files with updated values into the new version of files, MoR, on the other hand, writes updates to separate row-based delta log files. In other words, when we initially create an ingestion on MoR tables, it is written into columnar Parquet files, but any subsequent ingestions are stored in the delta log files. Depending on the lakehouse, these log files will be compacted into new parquet files either synchronously or asynchronously. Hudi uses **Apache Avro** to store these delta logs.

> **Note:** It's worth noting that with MoR, the entire files or partitions are not rewritten; instead, only the changes are written to new log files. This makes writing the changes much quicker, but reading the data requires additional effort.

|![Figure 3: Update on MoR table](/assets/images/posts/updates-on-mor-table.png){: width="100%" }|
|:-:|
|<sup>*Figure 3: Update on MoR table.*</sup>|<br/><br/>

In Apache Hudi, these delta logs are compacted into new columnar-based files either synchronously or asynchronously, and a new version of the columnar-based base file (Parquet) is generated in order to improve query performance. Frequent commits are possible with the MoR table type, but may not be possible with the CoW table type. Apache Iceberg demands additional maintenance services to carry out compaction tasks.

The following illustrates how Merge-on-Read works with a sequence of updates:

|![Figure 4: Merge-on-Read - How it works with a series of commits](/assets/images/posts/lakehouse-mor-how-it-works-illustration.png){: width="100%" }|
|:-:|
|<sup>*Figure 4: Merge-on-Read - How it works with a series of commits.*</sup>|<br/><br/>

#### Advantages and disadvantages of MoR

##### Advantages of MoR

- Write amplification is very low.

##### Disadvantages of MoR

While MoR has several advantages, it also comes with its own set of disadvantages:

- Higher read latency because MoR merges records in the base file (Parquet) and all its corresponding log files together on-the-fly during read operations, which can slow down query performance.
- Increased complexity. MoR can be more complex than CoR. It requires maintaining a separate set of index and base files to track changes, which can make the system more complicated and harder to manage.

## Optimistic concurrency control (OCC)

All lakehouses use optimistic concurrency control (OCC) to provide transactional guarantees between writers. In optimistic concurrency control, writers look for overlapping files, and if a conflict exists, they fail the operation and retry. OCC is a technique used to optimize concurrency by minimizing the need for locking and blocking concurrent transactions.

OCC relies on tracking the version or timestamp of each data item. When a transaction reads a data item, it notes the version or timestamp. When a transaction attempts to write or update a data item, it validates that the version or timestamp has not changed since the item was initially read. If the version and timestamp match, the write is permitted; otherwise, the transaction is rolled back (aborted) and must be retried.

## Incremental ingestion

todo

# Frequently asked questions (FAQ)

## What is table service in Apache Hudi?

