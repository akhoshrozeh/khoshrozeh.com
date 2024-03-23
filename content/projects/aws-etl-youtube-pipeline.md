+++
title = 'AWS ETL Youtube Pipeline'
date = 2023-12-25T09:02:25-05:00
draft = false
tags = ["AWS", "Lambda", "Athena", "Glue",  "ETL", "Python", "Big Data", "Data Pipeline"]
+++

### Tech Stack

*➔ Languages:* SQL, Python \
*➔ AWS Services:* S3, Glue, QuickSight, Lambda, Athena, IAM
*➔ SDK:* AWS Data Wrangler

### Introduction

In this project, we're simulating a client's needs for a data-driven campaign through an AWS data pipline. Such a task could be useful for a client who plans to advertise on Youtube and wants to get better insight on what factors can help make a video more popular.
SEE LEARNING GOALS
Questions to answer:
- How to categorize videos based on comments and statistics
- What factors affect how popular a Youtube video will be


### Goals
We will create:
    - Data Ingestion
    - Data Lake
    - AWS Cloud
    - ETL Design 
    - Scalability
    - Reporting

<!-- ### Learning Goals

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


### Big Data
- data can lose value over time
- data from 1 second ago vs data from 2 months ago or years
  - apps vs data scientist vs BI tools vs notebooks
    - 500 gb vs 10 TB
Challenges working with big data:
    - with so many different roles within a single production DB, it leads to a huge bottleneck -->


Setting up a new adminstrator role in AWS for this project. After configuring my AWS account in the CLI and downloading my [dataset](https://www.kaggle.com/datasets/datasnaek/youtube-new) from Kaggle, I'm going to create my S3 bucket which will serve as my data lake. Then I'm going to create the objects in S3 using the CLI and a couple shell scripts (see .sh files in Github). These objects follow Hive style partitioning. 

### Data Lake Architecture

We keep all our data in one centralized place, the data lake. From there, we would to like seamlessly move that data for different services, such as creating a relational DB, big data processing, ML, or maybe log analytics.

We want our data lakes to be scalable, performant and cost effective. We also want a unified governance. 

### Data Gravity
As data continues to grow, it becomes harder to move. That's why our data lake architecture is really important. We use AWS Glue as a layer around our data to help manage that complexity, in terms of accessing, security, and moving. It is a data catalogue. 

### AWS Glue: Catalog
What is it? \
It uses a crawler that goes through our datastores, and creates a table that infers our schema in the datastore.
Once it infers the schema, it creates the table in the Catalog. 
We use Glue because it allows us to have a data catalog. Learn more [here](https://docs.aws.amazon.com/whitepapers/latest/aws-glue-best-practices-build-efficient-data-pipeline/benefits-of-using-aws-glue-for-data-integration.html).

Glue creates the catalog/schema/table from our datastores using a crawler.

After we've created that catalogue, we can use Glue to also to create an ETL job.



### Creating the Crawler
We create the neccessary role and policy for the crawler allowing it to read the S3 bucket and access related services to Glue.
We create the crawler in Glue, but first create a database to store the results of the crawler. 
The crawler creates the data catalog in Glue so we can run SQL statements (via Athena) on it. 
We can view the results of crawler from the DB by using Athena to query it. 
There needs to somewhere to place the results of the query, so we create a new key in our S3 bucket.

However, Athena can't read our nested JSON. We need to build a simple ETL to transform the semi-structured JSON to structured DB.


### Building the Data Cleansing Pipeline: Semi-structured to structured data
Here's how the pipeline works at a high level:
- Raw data is stored in an S3 bucket as semi-structured JSON
- A lambda function that detects JSON from S3 and processes it using AWS Data Wrangler in Lambda Layer
- Processed data: converting JSON to Apache Parquet
- the processed data is then pushed to another S3 bucket and also cataloged in AWS Glue, making it available to queried by Athena via SQL statements.
  


Athena uses SerDe, which is a serializer and deserializer. 
So let's do the data cleansing by creating a light ETL that converts JSON into Apache Parquet
Parquet is columnar, meaning if only want a particular column, we don't have to read the entire row unlike in relational DBs like MySQL, which can be a bottleneck in 100 column table, for example. 


Why Lambda for the processing? Spark is too large of a framework for this instance i.e. too much overhead relative to the work for this project. Lambda is the best service for this use case. It's 
Lambda is a compute engine but not meant to process large amounts of data, like 100GB. It's not the correct service to process more than like 5GB of data.


### Building the ETl
In our data lake, we have S3 buckets for:
1. Landing Area (JSON, csv, etc. files)
2. Cleansed/Enriched (optimized into a format like Apache Parquet)
3. Deduped
4. Conformed

We get from stage 1 to 2 using Lambda. The code we'll upload to Lambda will be in Python and uses AWS Data Wrangler. AWS Data Wrangler is an open-source Python library developed by Amazon Web Services (AWS) that simplifies data preparation and integration with various AWS data services.


Wrangler is a framework that allows for simple interaction with AWS

First, I create the S3 nucket for the cleansed data. Then, create an IAM policy that allows the lambda function to read raw data from S3 and write the processed data back to another S3 bucket. Then , create the lambda function using Python3.

I added a custom lambda layer where the data wrangler dependency will reside. 