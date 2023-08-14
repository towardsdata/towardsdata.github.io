---
layout: post
title:  "How to perform a test for data completeness in Great Expectations"
kicker: "Data Quality"
subtitle: "In a data pipeline, comparing the upstream dataset (the preceding dataset) to the downstream dataset (the current dataset) is one way to ensure data completeness."
image: assets/images/posts-cover-images/data-completeness-gx.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-08-14 00:000:00 +0530
tags: [ "data-quality", "data", "data-observability", "great-expectations", "gx", "data-quality-dimensions" ]
categories: data-quality
featured: false
hidden: true
---

# Overview

This post focuses solely on how one can compare two datasets in Great Expectations using the Metric Store and Evaluation Parameters to ensure their completeness.

In a data pipeline, comparing the upstream dataset (the preceding dataset) to the downstream dataset (the current dataset) is one way to ensure data completeness.

For instance, we can compare:

- The number of rows between upstream and downstream datasets.
- The sum of the quantities e.g., `SUM(amount)` between upstream and downstream datasets

> **Expected values vs. observed values in Great Expectations:** As the name suggests, the expected values are those that are expected. The observed values, on the other hand, are the actual values from our data. GX compares expected values with observed values.

In order to validate data completeness by comparing upstream and downstream data, the upstream data's observed values must be stored somewhere in any data store. Great Expectations calls this data store a "metric data store" or "metric database". At the time of writing, Great Expectations only supports PostgreSQL databases as metric databases.

Before moving forward, let's first understand what **data quality dimensions** are.

## Data quality dimensions

We all know that data plays a crucial role in various aspects of modern business and decision-making processes. Therefore, "measuring" data quality is critical if we want to use it confidently. Measuring data quality enables organizations to extract the greatest amount of value from their data and make better choices based on it.

Without established guidelines, it can be challenging to consistently measure the quality of our data. Using data quality measurement guidelines or templates can facilitate and standardize data quality measurement, and one such guideline is **data quality dimensions**.

Data quality dimensions are specific aspects or characteristics used to assess or measure the quality of data. These data quality dimensions are useful guidelines for enhancing the quality of data assets. The number of data quality dimensions can vary depending on the framework or methodology used for data quality assessment. Several data quality dimensions are recognized; however, the **eight** most widely used data quality dimensions are:

- **Accuracy** - What degree of fact does a piece of information have?
- **Completeness** - The state of being complete and entire.
- **Consistency** - Does information stored in one place match relevant data stored elsewhere?
- **Timeliness** - Is our information available when it's needed?
- **Validity** - Invalid data affects the accuracy: Is information is a certain format, do business standards or rules apply to the information, or is it in an unusable format?
- **Uniqueness** - Ensures duplicate or overlapping data is identified and removed.
- **Integrity** - The maintenance of data relationships, which ensures referential and relational consistency.
- **Accessibility** - How easily authorized users can access, retrieve, and utilize the data.
  
## Validating between upstream and downstream datasets

The **Metric Store** and **Evaluation Parameter** features of Great Expectations enable us to store the upstream observed values and read them, respectively, during the downstream data validation stage. Note that this Metric Store and Evaluation Parameter are still experimental features of Great Expectations, and we expect configuration and capability to evolve rapidly.

Using the Metric Store feature, as depicted below, any required observed values that we would like to store for future reference can be stored in our metric database. In subsequent phases of our data pipeline, we can perform a completeness check by comparing the stored values from the metric database with our current dataset.

|![Figure 1: Comparing upstream and downstream datasets using Metric Store and Evaluation Parameter features](/assets/images/posts/compare-upstream-data-against-downstream-data.png){: width="100%" }|
|:-:|
|<sup>*Figure 1: Comparing upstream and downstream datasets using Metric Store and Evaluation Parameter features.*</sup>|<br/><br/>
