---
layout: post
title:  "AWS Lake Formation"
kicker: "AWS Lake Formation"
subtitle: "Information is only valuable if it is of high quality. It's focused on making sure that the data complies with our data quality dimensions."
image: assets/images/posts-cover-images/lake-formation.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-08-02 00:000:00 +0530
tags: [ "lake-formation", "data-lake", "aws-lake-formation", "secured-data-lake" ]
categories: lake-formation
featured: false
hidden: true
---

# What is AWS Lake Formation?

AWS Lake Formation is a *fully managed service* that makes it easy to build, *secure*, and *manage* data lakes. To put it simply, *it's a secured data lake*. Many of the complex manual steps that are usually required for creating data lakes are made simpler and automated by lake formation. These steps include *gathering*, *cleansing*, *transferring*, and *cataloging* data as well as securely making that data available for analytics and machine learning.

Lake Formation has its *own* permissions model that works *along with* with the IAM permissions model. This permissions model is defined *centrally*, which enables fine-grained access to data stored in data lakes through a simple grant or revoke mechanism, much like an RDMS. Lake Formation permissions are enforced using *granular controls* at the column, row, and cell levels across AWS analytics and machine learning services, including Amazon Athena, Amazon QuickSight, and Amazon Redshift.

> **Lake Formation vs. Data Lake:** A data lake is a centralized data repository that holds all of our data, both structured and unstructured, of any size. Lake Formation, on the other hand, is a fully managed, *secure data lake*.

# How does Lake Formation work?

Having said that above, the fully managed AWS Lake Formation service allows us to build a secure data lake in a matter of days with minimal effort and time. We first need to identify existing data stores in Amazon S3 or relational and NoSQL databases and move them into our data lake. Following that, crawl, catalog, and get the data ready for analytics. Lastly, provide our consumers with safe self-service access to the data using the analytics services of their choice.

Having said that, at a high level, the three tasks outlined below should be carried out:

- **Data ingestion and management**
- **Security management**
- **Data Sharing**

## Data ingestion and management

### Data ingestion

Once we identify the locations of our existing databases (whether they are on AWS or in external data sources) and give access credentials, Lake Formation reads the data and its metadata (schema) to understand the information contained in the data source. The data is subsequently imported into our new data lake, and the metadata is stored in a *central* catalog. Typically, the data that is being ingested from multiple data sources ends up in S3 buckets of our choice. Note that both bulk and incremental data ingestions are supported.

|![Figure 1: Data Flow](/assets/images/posts/data-sources-to-lake-formation-to-s3-buckets.png){: width="60%" }|
|:-:|
|<sup>*Figure 1: Data Flow.*</sup>|<br/><br/>

### Catalog and label our data

We can use **AWS Glue crawlers** to read our data in Amazon S3, extract table definitions, and store them in a searchable AWS Glue Data Catalog. After that, make use of Lake Formation **tag-based access control** (**TBAC**) to manage permissions on databases, tables, and columns.  

> **AWS Glue Crawler:** A AWS Glue Crawler examines our data store, extracts metadata (which contains the format, database and table schema, and associated properties of the data), and stores it in the AWS Glue Data Catalog.

### Security management

