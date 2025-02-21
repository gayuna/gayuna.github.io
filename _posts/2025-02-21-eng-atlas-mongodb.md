---
title: The Life of MongoDB - Starting with the Free Tier on Atlas
categories:
  - Technologies
tags:
  - Atlas
  - MongoDB
  - DocumentDB
---

#### The Natural Choice: Using a Document Database

When choosing a database to store students' homework at DangwooMath, we decided to use MongoDB. Some students submit a 3-minute homework video, while others submit an hour-long one. The audio from these videos is extracted and stored as text, and once stored, it doesn’t require updates. Since each homework submission is independent, relationships between entries are not important, and only simple queries will be needed. Additionally, because we need to store text of varying lengths, using a Document DB was the natural choice.

#### Getting Started with Atlas

At this point, we decided to use Atlas. Atlas is a service directly operated by MongoDB that allows seamless use of MongoDB in cloud environments like AWS, Google Cloud, and Azure. Since I am solely responsible for the backend, using a fully managed SaaS solution seemed more efficient than setting up a server and configuring the database myself. Also, Atlas provides a free tier (512MB), making it ideal for running a proof-of-concept (PoC) before scaling up.

After signing up on the [Atlas website](https://cloud.mongodb.com/), the first step is to create a cluster. The first cluster can be created using the Free Tier.

![Create Cluster](https://github.com/user-attachments/assets/47bb938e-75f1-45ae-a2af-4df8ec336c42)

After naming the cluster appropriately, choose a cloud provider among AWS, Google Cloud, or Azure. I selected AWS and kept the default recommended region, which appears to be suggested based on location.

If you check the Preload sample dataset option before creating the cluster, MongoDB will initialize it with sample data, allowing you to practice queries and other tasks. I enabled this option during setup and later manually deleted the data before deploying the actual service. If you are new to MongoDB, I highly recommend using this feature.

Once the cluster is created, a database user account and password are automatically generated. You can use them as-is or modify them before clicking the Create button to finalize the access credentials. Be sure to store this information securely in your project’s .env file or another safe location. (Of course, you can always create a new account or change the password later.)

#### Connecting to Atlas

During development, I used [pymongo](https://www.mongodb.com/ko-kr/docs/languages/python/pymongo-driver/current/) for database interactions and the [MongoDB extension](https://www.mongodb.com/products/tools/vs-code) for VS Code to visualize and manage data.

Clicking Connect in the cluster dashboard presents several options for connecting to the database. To integrate it into your code, select Drivers.

![Connect-drivers](https://github.com/user-attachments/assets/b6ba1873-b885-4d05-907b-92a6aa0342be)

Installation instructions and sample code for various programming languages are provided. If you toggle View full code sample, you can access a complete script that can be run locally for direct testing.

Additionally, if you select MongoDB for VS Code instead of Drivers, you will be guided through connecting the MongoDB extension to VS Code. This extension allows quick data queries, supports JavaScript-based query testing, and provides a Playground feature that can convert queries into different programming languages. If you use VS Code, I highly recommend leveraging this extension.

Meanwhile, non-developers also needed an environment where they could view data. Instead of developing a new interface from scratch, I guided them to use MongoDB Compass, the official GUI tool provided by MongoDB. Naturally, it’s best to issue a read-only account for such use cases to ensure security.

![Add access](https://github.com/user-attachments/assets/1ddc9e38-1833-4a08-9a54-dbadf8d04852)

Go to Security and navigate to the Database Access section, then select Add new database user.

![role](https://github.com/user-attachments/assets/0455e6b2-ebec-4a2a-ab81-dedefcd27cc5)

At this stage, you can easily create a read-only account by selecting Only read any database from the Built-in Role options. This ensures that users can view data without making any modifications.

#### MongoDB deep dive

##### MongoDB Hierarchy

The structure of MongoDB can be explained as follows:

```text
Cluster
 ├── (Shards)
 │    ├── (Replica Set)
 │    │    ├── (Instance)
 │    │    │    ├── Database
 │    │    │    │    ├── Collection
 │    │    │    │    │    ├── Document
 ```

 The cluster we created earlier is at the highest level, and within it, data can be sharded and replicated. In the diagram above, Shards, Replica Sets, and Instances represent infrastructure-level concepts, but in the free tier, these configurations cannot be adjusted manually.

The free tier only allows M0 clusters, which do not support sharding. Additionally, the Replica Set is automatically configured with one Primary and two Secondary nodes, and this setup cannot be modified. (Sharding is a powerful scalability feature of MongoDB and should be considered when handling large-scale expansions, but this article will not cover it.)

When developing applications in the free tier, the first step is creating a Database.

A Database contains one or more Collections. Similar to relational databases (RDBs), databases exist within an instance, but MongoDB does not support references between different databases.

A Collection is analogous to a Table in RDBs, and each Document is stored within a collection. A Document is similar to a Row in a table, containing actual data and stored in a BSON (Binary JSON) format. Since BSON is converted into machine code, it requires a transformation process before storage and retrieval.


##### Aggregating Data in MongoDB

One of MongoDB’s strengths is its fast read speed. To leverage this advantage, MongoDB not only supports simple CRUD operations but also provides comprehensive information on the [Aggregation Pipeline in its official documentation]((https://www.mongodb.com/ko-kr/docs/manual/core/aggregation-pipeline/)), which enables data aggregation and transformation. Aggregation is performed by chaining one or more stages together. This can be executed in code as well as through the UI provided by Atlas (both web and Compass).

![pipeline](https://github.com/user-attachments/assets/ded0e66b-ab9d-47a6-8411-ef705e321ad4)

As you add stages to the pipeline in the UI, you can view the output of each stage in real time, allowing you to build and refine the aggregation process interactively. Once the pipeline is complete, you can click Run, and then Export to download the results as a JSON or CSV file. Additionally, you can save the pipeline for future reuse. The UI also includes a feature that converts the pipeline into code for different programming languages. Notably, [if you choose to Save the pipeline as a View, it remains in a read-only state, always displaying data with the applied aggregation]((https://www.mongodb.com/ko-kr/docs/compass/current/views/)). This feature is particularly useful for enabling non-developer team members with read-only access to independently view the processed data.

##### Creating Indexes in MongoDB

Just like in SQL, MongoDB allows you to create indexes to speed up search operations. It supports basic single-field and compound indexes, as well as specialized indexes for geospatial data ([Geospatial Indexes](https://www.mongodb.com/ko-kr/docs/manual/core/indexes/index-types/index-geospatial/)). Beyond the free plan, you can also use these indexes as shard keys to distribute data in a sharded cluster using [Hashed Sharding](https://www.mongodb.com/ko-kr/docs/manual/core/hashed-sharding/#std-label-sharding-hashed-sharding).

However, MongoDB inherently prioritizes fast read performance at the cost of slower write and update operations. Excessive use of indexes can increase operational costs significantly. Therefore, it is crucial to apply indexes selectively, focusing on frequently used queries to maintain a balance between performance and cost.

So far, we’ve covered the basic concepts of MongoDB in the context of using Atlas. If you want to dive deeper into MongoDB, I highly recommend checking out [MongoDB University](https://learn.mongodb.com/), where MongoDB provides high-quality courses to help you expand your knowledge.
