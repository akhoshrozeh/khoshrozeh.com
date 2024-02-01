+++
title = 'AWS ETL Youtube Pipeline'
date = 2023-12-25T09:02:25-05:00
draft = true
tags = ["AWS", "Lambda", "Athena", "Glue",  "ETL", "Python", "Big Data", "Data Pipeline"]
+++

### Tech Stack

*➔ Languages:* SQL, Python3 \
*➔ Services:* AWS: S3, Glue, QuickSight, Lambda, Athena, IAM
*➔ SDK:* AWS Data Wrangler

### Introduction

In this project, we're working backwards by simulating a customer's needs for a data-driven campaign. 

Questions to answer:
- How to categorize videos based on comments and statistics
- What factors affect how popular a Youtube video will be


### Goals and Success Criteria
How to measure success?
We will create:
    - Data Ingestion
    - Data Lake
    - AWS Cloud
    - ETL Design 
    - Scalability
    - Reporting

### Learning Goals

- How to build a data lake from scratch in AWS S3
  - Joining semi-structured data and structured data
- Lake House architecture design
  - Best practices for cost and performance
- Data Lake vs. Data Warehouse
- Data Lake design in layers, partitioned for cost-performance
- AWS Data Catalogue
- Designing and Developing ETL in AWS Glue using Spark Jobs
- Amazon SNS for Alerting
- Using Amazon Athena to run SQL statements against datasets
  - Impact of querying optimized data layers vs non-optimized data layers
- Ingest changes incrementally and schema evolution
- Business Intelligence Dashboards using AWS Quicksight


### big Data
- data can lose value over time
- data from 1 second ago vs data from 2 months ago or years
  - apps vs data scientist vs BI tools vs notebooks
    - 500 gb vs 10 TB
Challenges working with big data:
    - with so many different roles within a single production DB, it leads to a huge bottleneck



### Data Lake Architecture

We keep all our data in one centralized place, the data lake. From there, we would to like seamlessly move that data for different services, such as creating a relational DB, for big data processing, ML, or maybe log analytics.

We want our data lakes to be scalable, performant and cost effective. We also want a unified governance. 

### Data Gravity
As data continues to grow, it becomes harder to move. That's why our data lake architecture is really important. We use AWS Glue as a layer around our data to help manage that complexity, in terms of accessing, security, and moving. It is a data catalogue. 

### AWS Glue Catalog
What is it? \
It uses a crawler that goes through our datastores, and creates a table that infers our schema in the datastore.
Once it infers the schema, it creates the table in the Catalog. 


After we've created that catalogue, we can use it for an ETL process and then put it back in another datastore. 

We can also run SQL statements on that catalog using AWS Athena!

### Creating the Crawler
We create the crawler in Glue, but first create a database to store the results of the crawler. 
The crawler creates the data catalog in Glue so we can run SQL statements (via Athena) on it. 
We can view the results of crawler from the DB by using Athena to query it. 
There needs to somewhere to place the results of the query, so we create another S3 bucket.

However, Athena can't read our JSON. We need to build a simple ETL to transform the semi-structured JSON to structured DB.

Athena uses SerDe, which is a serializer and deserializer. 
So let's do the data cleansing by creating a light ETL.
JSON -> Apache Parquet
Parquet is columnar, meaning if only want a particular column, we don't have to read the entire row unlike in relational DBs like MySQL, which can be a bottleneck in 100 column table, for example. 

### Building the Data Cleansing Pipeline: Semi-structured to structured data
JSON => AWS S3 Raw Bucket <= AWS Lambda (JSON to Apache Parquet)


Tools:
- AWS Lambda
- Data Wrangler
- Pandas

Why Lambda? Spark is too large of a framework for this instance. Lambda is the best service for this use case. It's 
Lambda is a compute engine but not meant to process large amounts of data, like 100GB. It's not the correct service to process more than like 5GB of data.
limits of lambda:
    - 10gb of memory
    - 50mb deployment package
    - 6 vCPUs
    - 15 min timeout

In our data lake, we have S3 buckets for:
1. Landing Area (JSON, csv, etc. files)
2. Cleansed/Enriched (optimized into a format like Apache Parquet)
3. Deduped
4. Conformed

We get from stage 1 to 2 using Lambda. The code we'll upload to Lambda will be in Python and uses AWS Data Wrangler. AWS Data Wrangler is an open-source Python library developed by Amazon Web Services (AWS) that simplifies data preparation and integration with various AWS data services.