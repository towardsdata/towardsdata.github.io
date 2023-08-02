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
