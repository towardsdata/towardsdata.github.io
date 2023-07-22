---
layout: post
title:  "Great Expectations takes great care of our data!"
kicker: "Data Quality"
subtitle: "Information is only valuable if it is of high quality. It's focused on making sure that the data complies with our data quality dimensions."
image: assets/images/posts-cover-images/data-quality.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-07-22 00:000:00 +0530
tags: [ "data-quality", "data", "data-observability", "great-expectations", "gx", "ge" ]
categories: data-quality
featured: false
hidden: true
---

# Data quality

Information is only valuable if it is of high quality. How can we get data of such high quality, then?  The answer is simple: testing the data quality is what assures high quality. We know the "what" part of it gets us quality, but the "how" part is much more crucial in producing high-quality data. This post covers in depth how to ensure the integrity of data. Only high-quality data will produce accurate analysis, which in turn can lead to reliable business decisions. Data quality may be simple to recognize but challenging to quantify.

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
  
## Testing data

Similar to unit testing in software engineering, data testing has to become a regular practice in data engineering. The organization may establish a **data acceptance test** that restricts the use of data unless its owners can demonstrate that it satisfies the organization's quality standards.

### Data quality testing stages

1. **First stage:** The first place that we need to test data is **at the point of ingestion**. Before ingesting ingested data from its source to its target destination, we want to ensure that all of the data has been successfully validated.
2. **Second stage:** The second place that we need to test data is **at the point of transformation**. During transformation, we would need to check pre- and post-conditions, such as:
   - Null checks
   - Valid value checks
   - Schema checks
   - Referential integrity checks, and so on.
  
### Data quality tools

There are various data quality tools—both commercial and open source—that are currently on the market. This blog focuses on one of the hand-picked open source data quality tools called **Great Expectations**, among other open source tools.

We've taken into account the following factors while evaluating the open source data quality tools:

- Is the tool **able to deal with most widely used data quality dimensions**?
- **How easy is it for both data engineers and data analysts** to learn how to write data quality checks or tests and get good at them quickly?
- **How well the documentation and API guides** are supported?
- **How flexible and extensible** the tool is
- **How active and responsive is the respective community in Slack**? In general, Slack is considered to be far more helpful than submitting our questions elsewhere. We could ask questions and receive straight answers from committers, which is one of the best things about Slack.
- The **rate at which the tool is evolving and gaining new capabilities**.
- Last but not least, which one of them is **more widely used across enterprises**?

Let's take a closer look at **Great Expectations** to see how it might assist us in obtaining reliable data.

---

# Introduction to Great Expectations (GX)

Great Expectations, shortly referred to as **GX**, is a **powerful** and **flexible** open-source data quality solution on the market today. We'll take a close look at it with examples to demonstrate how powerful and flexible GX tool is.

Unlike traditional unit tests, GX applies tests to data instead of code. To put it simply, **in GX, testing is performed on data rather than code**. It's a Python library that enables us to verify that our data is accurate by **profiling**, **validating**, and **documenting** it. 

> **Profiling**, or **data profiling**, is the process of examining, analyzing, and creating useful summaries of data that aid in the discovery of data quality issues.

Based on our analysis of the open source software (OSS) data quality offerings, we've concluded that Great Expectations stands out among the rest. However, it might be difficult or perhaps daunting to comprehend at first, even for seasoned developers, but it gets easier and easier the more we work on it. Great Expectations comes with many building blocks and concepts, and there are various ways to setup and execute this tool. Due to many design components and concepts, this tool demands time and effort to fully comprehend. The most perplexing aspect is that we can configure and deploy this tool in a variety of ways, depending on our preferences and needs, so we may be uncertain of where to begin and which approach we should use! Note that the GX team and the community are working hard to make things simple and straightforward.

This post saves the day. We make every effort to bring you up to speed on Great Expectations, including the best approaches. We do not cover all of Great Expectations' features and procedures, but we do go over the essential steps that must be taken in order to set up and execute Great Expectations as well as how to reuse the Expectations Suite and Checkpoints. For more details, please check out Great Expectations's online [documentation](https://docs.greatexpectations.io/docs/){:target="_blank"}.

Before diving headfirst into the Great Expectations tool, we should be aware of its known issues and limitations!

## Issues and limitations

We have compiled a list of known issues and limitations with Great Expectations.

### Limitations

#### Lack of referential integrity checks

While Great Expectations doesn't have built-in support for referential integrity checks, we can achieve referential integrity validation by leveraging its capabilities in combination with custom code.

Here's a high-level approach to implementing referential integrity checks in Great Expectations:

- **Write Custom Expectations:** Write custom expectations to perform referential integrity checks. We can define expectations that verify the relationships between foreign keys and primary keys across different tables. For example, we can create an expectation that checks if all foreign key values in a column exist in the referenced primary key column of another table.
- **Create Custom Validators:** If our referential integrity checks require more complex logic or involve multiple tables, we may need to create custom validators using Great Expectations' API. Custom validators allow us to define advanced validation rules and implement complex checks beyond what the built-in expectations offer.

Note that the exact implementation of referential integrity checks may vary based on our specific data sources and requirements. We may need to write custom code using the Great Expectations API or leverage additional tools and libraries to accomplish more complex referential integrity validations.

#### Validating multiple data files in a single request

GX validates one data file at a time! Note that Great Expectations (GX) currently does not support validating multiple data files from a directory or S3 bucket using a single batch request. Great Expectations is designed to work with individual files or datasets rather than entire directories. This doesn't indicate that GX is incapable of validating multiple files within a directory or S3 bucket. GX can validate multiple files, but each must be submitted as a separate batch request, which can be a hassle.

#### Windows support

The Great Expectations team's statement about Windows support is as follows:

*"Great Expectations is developed and tested on macOS and Linux Ubuntu. Windows support for the open source Python version of GX is currently unavailable. If you’re using GX in a Windows environment, you might experience errors or performance issues."* 

Refer to the following link: https://docs.greatexpectations.io/docs/guides/setup/installation/local/#prerequisites
