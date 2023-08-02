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

|![Figure 1: Data Flow](/assets/images/posts/data-sources-to-lake-formation-to-s3-buckets.png){: width="70%" }|
|:-:|
|<sup>*Figure 1: Data Flow.*</sup>|<br/><br/>

### Catalog and label our data

We can use **AWS Glue crawlers** to read our data in Amazon S3, extract table definitions, and store them in a searchable AWS Glue Data Catalog. After that, make use of Lake Formation **tag-based access control** (**TBAC**) to manage permissions on databases, tables, and columns.  

> **AWS Glue Crawler:** A AWS Glue Crawler examines our data store, extracts metadata (which contains the format, database and table schema, and associated properties of the data), and stores it in the AWS Glue Data Catalog.

### Security management

#### Define and manage access controls

Lake Formation provides a single place (centrally) to manage access controls for data in our data lake. We can define security policies that restrict access to data at the database, table, column, row, and cell levels. These policies apply to IAM users and roles, as well as to users and groups that are federated through an external identity provider. 

We can use fine-grained controls to access data protected by the Lake Formation within:

- **Amazon Redshift Spectrum**
- **Athena**
- **AWS Glue ETL**
- **Amazon EMR for Apache Spark**

#### Audit logs

Lake Formation provides extensive audit logs with CloudTrail to monitor access and demonstrate compliance with centrally defined policies. We can audit the data access history in our data lake via Lake Formation. This enables us to identify which users or roles have tried to access which data, via which services, at what time. We can access audit logs in the same way we access any other CloudTrail logs using the CloudTrail APIs and console.

#### Row and cell-level security

Data filters offered by Lake Formation enable us to restrict access to a set of columns and rows. Use row and cell-level security to safeguard sensitive data, such as personally Identifiable Information (PII).

We can create the following levels of data security using Lake Formation's data filtering capabilities:

##### Column-level security

Column-level security allows users to view only specific columns that they have access to in the table.

##### Row-level security

Row-level security allows users to view only specific rows of data that they have access to in the table.

##### Cell-level security

Cell-level security combines row filtering and column filtering for a highly flexible permissions model. If we think of the rows and columns of a table as a grid, we can restrict access to individual elements (cells) of the grid anywhere in the two dimensions by using cell-level security.

#### Tag-based access control (TBAC)

Use Lake Formation *tag based access control* to manage hundreds or even thousands data permissions by creating custom labels called LF-tags. We can define LF-tags and attach them to databases, tables, or columns. Then, share controlled access across analytic, ML, and ETL services for consumption. LF-tags ensure that data governance can be scaled easily by replacing the policy definitions of thousands of resources with a few logical tags. Users can easily find the information they need for analysis by using the text-based search function that Lake Formation offers over this metadata.

#### Cross account access

Lake Formation permission management capabilities make it easier to secure and manage distributed data lakes *across different AWS accounts* through centralized, fine-grained control over the Data Catalog and Amazon S3 locations.

### Data sharing

The data sharing capability enables us to set up permissions on datasets stored in different data sources, such as Amazon Redshift, *without migrating data or metadata* into Amazon S3 or AWS Glue Data Catalog.

## How it works

In Lake Formation, we can implement permissions on two levels:

- **Metadata layer:** Applying metadata-level restrictions to databases and tables that are part of the Data Catalog.
- **Storage layer:** Managing storage access permissions on the underlying data stored in Amazon S3 on behalf of integrated analytical engines such as Amazon Athena, AWS Glue, Amazon EMR, or Amazon Redshift Spectrum.

### Lake Formation permissions management high-level workflow

todo

# Frequently asked questions (FAQ)

## What is credential vending?

The term "vending" literally refers to earning income by selling goods. Lake Formation allows third-party services to integrate with Lake Formation by using credential vending API operations.

## What is a data catalog?

In general, the Data catalog is a persistent repository for storing metadata. A data catalog in AWS is referred to as **AWS Glue Data Catalog** or simply Data Catalog, and it is a managed service that enables us to *store*, *annotate*, and *share* metadata in the AWS Cloud. It provides a *centralized repository* where disparate systems, such as any data store, etc., can store and find metadata to locate Underlying data in data silos and then use that metadata to query and transform the data.

AWS Lake Formation uses the AWS Glue Data Catalog to store and retrieve metadata about data lakes. Lake Formation provides a hierarchy of permissions to control access in the AWS Glue Data Catalog.

## How many Data catalogs can exist per AWS region and per AWS account?

Each AWS account has *one* Data Catalog per AWS Region.

## What is Principal in AWS?

A principal is an AWS IAM user or role or an Active Directory user.

## Who is the Data Lake administrator?

A data lake administrator is a *principal* who can grant any permission on any Data Catalog resource or data location to any other principal (including self). Data lake administrator can then grant more granular permissions of resources to other principals. Note that IAM administrative users—users with the `AdministratorAccess` AWS managed policy—are not automatically data lake administrators. 

## What is Blueprint in Lake Formation?

A blueprint is a data management template that enables us to easily ingest data into a data lake. Lake Formation provides several blueprints, each for a predefined source type, such as a relational database or AWS CloudTrail logs. From a blueprint, we can create a **workflow**.

## What is a workflow?

A workflow is a container consisting of **AWS Glue crawlers**, **jobs**, and **triggers** that are generated to orchestrate the loading and updating of data. create the workflow in Lake Formation, and it executes in the AWS Glue service. Lake Formation can track the status of a workflow as a single entity. 

Blueprints use the data source, the data target, and the schedule to configure the workflow. In other words, when we define a workflow, we select the blueprint upon which it is based. We can then run workflows on demand or on a schedule. Workflows that we create in Lake Formation are visible in the AWS Glue console as a directed acyclic graph (DAG). Using the DAG, we can track the progress of the workflow and perform troubleshooting.

## What is tag-based access control in the AWS Lake Formation, and what's the need for it?

As the number of tables and users increase, data stewards and administrators are looking for ways to manage permissions on data lakes easily at scale. Customers are struggling with “role explosion” and need to manage hundreds or even thousands of user permissions to control data access. For example, for an account with 500 resources and 50 principals, the data steward would have to create and manage up to 2500 policy statements. As new principals and resources get added or deleted, these policies must be updated to keep the permissions current.

Lake Formation **tag-based access control** (**TBAC**) solves this problem by allowing data stewards or data administrators to create **LF-tags** (based on their business needs) that are attached to resources. Rather than specifying policies on named resources, policies can be created on a smaller number of logical tags. In other waords, we can create and manage policies with tens of logical tags instead of the thousands of resources. LF-tags enable us to categorize and explore data based on *taxonomies*, which reduces policy complexity and scales permissions management. 

> **Note:** Taxonomy, in this context, is a *data taxonomy* is a way of organizing and classifying data. It involves creating a hierarchy of categories and subcategories that can be used to classify and organize data consistently and logically, allowing datasets to be understood quickly regardless of who is viewing them.

Lake Formation TBAC decouples policy creation from resource creation, allowing data administrators to manage permissions on many databases, tables, and columns without having to update policies every time a new resource is added to the data lake. TBAC allows us to create policies even before the resources come into existence. We can now define LF-tags, associate them at the database, table, or column level, and share controlled access across analytic, machine learning (ML), and ETL services for consumer consumption.

### What values can be assigned LF-tags?

- Tags can be up to 128 characters.
- Values can be up to 256 characters.
- Up to 15 values per tag.
- Up to 50 LF-tags per resource.
  
Examples: 
Key=Confidentiality | Values=private, sensitive, public
Key=Project | Values=project1, project2