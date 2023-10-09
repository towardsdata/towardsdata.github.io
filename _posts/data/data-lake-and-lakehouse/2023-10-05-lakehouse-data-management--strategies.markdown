---
layout: post
title:  "Lakehouse's data management strategies"
kicker: "Lakehouse's data management strategies"
subtitle: "The modern data lake is referred to as a 'lakehouse', which has made the decision to employ a hybrid architecture that combines the features and advantages of a data lake and a data warehouse to address the shortcomings and challenges of data management associated with the data lake."
image: assets/images/posts-cover-images/lakehouse-data-management.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-10-05 00:000:00 +0530
tags: [ "storage-strategies", "copy-on-write", "merge-on-read", "cow", "mor", "lakehouse" ]
categories: lakehouse
featured: false
hidden: false
---

# Overview

Let's go over in depth in this post how modern data lakes perform **deletes** and **upserts**, much like OLTP databases. An "upsert" is a mix of "update" and "insert," and it refers to the process modifying an existing record in a data store if it already exists or adding a new record if it doesn't. Updates and deletions are fundamental operations in any OLTP database. These OLTP databases use row-level storage with built-in indexes, which makes it faster to locate a particular record for upserts. 

> **Indices vs. indexes**: Both "indexes" and "indices" are acceptable plural forms of the word "index".

There is no built-in support for upserts in a traditional Hadoop, aka Data Lake, which comprises the Hadoop Distributed File System (HDFS) and a number of data processing frameworks including Apache MapReduce, Apache Spark, and Apache Hive. Data Lake's primary use case was to process and analyze a huge volume of data in batch mode. Since real-time deletes and upserts are not naturally suited to this form of operation, ACID (Atomicity, Consistency, Isolation, Durability) transactions were not intended to be supported by it, and it wasn't part of the design objective.

In traditional Data Lakes, the Apache Spark approach reads in and overwrites the whole table or partition with each update to handle batch updates even for small change. We are all well aware how bad it was.

Modern Data Lakes have emerged to solve the ACID feature's transaction abilities. The modern Data Lake is known as a "lakehouse", which has chosen a hybrid architecture that combines the features and advantages of a data lake and a data warehouse. There are three popular lakehouses at the present time: **Apache Hudi**, **Apache Iceberg**, and **Delta Lake**. The primary features of the data warehouse used in lakehouse are indexing and ACID transactions. Indexing and ACID transactions are critical for today’s lakehouse. For instance, some important use cases, such as data retention and change data capture (CDC), rely heavily on ACID transactions.

# Apache Parquet is the preferred storage file format

Lakehouse has chosen **Apache Parquet** as the preferred storage file format. For a number of reasons, Apache Hudi, Apache Iceberg, and Delta Lake use the Apache Parquet file format in their efforts to offer stability, performance, and and ACID transaction support on top of data lakes:

* **Columnar storage**: Parquet is a *columnar storage* file format, which means that data is stored column by column rather than row by row. This columnar storage layout is highly efficient for analytical queries because it allows for predicate pushdown and compression, resulting in reduced I/O and improved query performance.
* **Efficient compression**: Parquet supports various compression algorithms, which can significantly reduce the storage footprint and improve query performance. Columnar storage further enhances compression by grouping similar data together.
* **Schema evolution**: Schema evolution is a feature that enables us easily change a table's current schema to accommodate data that is evolving over time without affecting existing data.
* **Column pruning**: Columnar storage can be a good choice when queries do not read all columns of the data. This is known as column pruning and may result in a significant speed boost.

# Lakehouse's data management strategies

Apache Hudi, Delta Lake, and Apache Iceberg are three popular open-source projects that aim to improve the management and processing of data in data lakes. They each have their own approach to handling data updates or deletes, and differing design choices relate to the concepts of **Copy on Write** (**CoW**) and **Merge on Read** (**MoR**). In other words, these Lakehouse architectures often combine CoW and MoR strategies, depending on the specific needs and design principles of the framework. For instance, in Apache Hudi, Copy on Write (CoW) is the default storage technique whereas Merge on Read (MoR) is an optional one.

Let's examine each of these approaches in further detail by using one of these Lakehouse frameworks to see how it applies these strategies to dealing with updates and deletions. We'll make use of the Apache Hudi framework to walk through these strategies, but a similar idea can apply to Delta Lake and Apache Iceberg.

## How does Apache Hudi perform updates and deletes under the hood?



