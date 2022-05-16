---
title: Real-time data pipeline with Debezium and Snowflake Part 1
tags:
  - data
date: 2022-05-16 18:33:20
---


Data is everywhere. Behind every dashboard and machine learning model is data. However, before value can be derived from data in the form of insights or a predictive model, it must be integrated from various sources and brought into a data warehouse. The data warehouse then serves as the single point of contact for various teams and their data needs. In this series of posts we will look at building a real-time data pipeline using Debezium and Snowflake. We will begin by taking a look at the overall architecture of the data pipeline and then proceed to build it using Docker.

## What is a data pipeline?  

Data pipelines are the machinery which move the data from various sources (such as operational databases, REST APIs) to a destination (such as a data warehouse). They help lay the foundation for analytics, and machine learning. Data pipelines can be as simple as loading data from a REST API into a SQL table in the warehouse or they can be more complex and involve multiple stages of extracting, cleaning, and transforming data.   

Data pipelines can be either batch or real-time. Batch processing involves moving data from the source to the destination periodically. In contrast, real-time processing involves moving data as soon as it is generated in the source systems.  

There are various open-source, and commercial tools available for building pipelines. For this series of posts we will use open-source tools Debezium and Kafka to build the data pipeline. 

## What is Debezium?  

Debezium is an open-source platform for change data capture. Change data capture, or CDC for short, is the process of capturing changes in the database and replicating them to a destination such as a data warehouse. The way in which change data is captured varies from one database to another. For example, MySQL uses binlog to store change data. Debezium allows us to capture this and write it to Kafka without having to worry about the underlying database.  

Debezium operates by running as connectors within a Kafka Connect cluster.

## What is Kafka Connect?  

Kafka Connect is a framework for connecting Kafka to external systems such as databases. It works by running connectors which write data to Kafka or read data from it. As you will see in the course of this series, Kafka Connect operates as a cluster separate from Kafka itself. When we start building the pipeline using Docker we will create and configure a Kafka Connect cluster to run our connectors. 

Connectors are created by submitting configuration to the cluster using a REST API. We will run a Debezium source connector to read data from a MySQL database and a Snowflake sink connector to write data to Snowflake.

## What is Snowflake?  

Snowflake is a cloud data warehouse provided as a SaaS. It allows you to store, process, and analyse vast amounts of data by provisioning data warehouses to handle the scale. Through out this series of posts we will use Snowflake as the destination for all our data.

## The big picture

Before we get busy with building the pipeline let's take a look at the blueprint.  

Whenever data is inserted, updated, or deleted in MySQL it is written to the binlog. The binlogs are files which store these events. Our goal is to consume these binlogs and create a replica of the table in Snowflake.   

The first step is the consumption of these logs. We will do this by starting a Debezium source connector in the Kafka Connect cluster. It will read the binlogs and generate a JSON object in Kafka for every event in the binlog. The configuration of the connector will specify which table to read and in which Kafka topic the data will go.  

Once the data is in Kafka, the next step is to write it to Snowflake. We will start a Snowflake sink connector which will read data from Kafka and write it to a table in Snowflake. Since Debezium will generate events for every insert, update, and delete operation, we will do some magic in Snowflake to make the destination table look exactly like the source table in MySQL.  

The flow of data looks as follows:

{% raw %}
<center> 
    MySQL → Source Connector → Kafka → Sink Connector → Snowflake 
</center>
{% endraw %}  

## Conclusion  

Replicating data from transactional databases into a warehouse allows you to create a single place to access all your data. This enables you to build intelligence in the form of analytics, and machine learning. Open-source tools like Debezium and Kafka allow you to replicate data as soon as it is generated and thus reduce the time it takes for the data to be available to derive insights.  

In the next post we will begin building the pipeline by setting up MySQL, Kafka, and the Kafka Connect cluster.   

Finito.