---
title: Creating a realtime data platform with Pinot, Airflow, Trino, and Debezium
tags:
  - architecture
date: 2024-06-20 13:19:19
---


I'd previously written about creating a [realtime data warehouse with Apache Doris and Debezium](/2023/11/04/Creating-a-data-warehouse-with-Apache-Doris/). In this post we'll see how to create a realtime data platform with [Pinot](https://pinot.apache.org/), [Trino](https://trino.io/), [Airflow](https://airflow.apache.org/), [Debezium](https://debezium.io/), and [Superset](https://superset.apache.org/). In a nutshell, the idea is to bring together data from various sources into Pinot using Debezium, transform it using Airflow, use Trino for query federation, and use Superset to create reports.

## Before We Begin  

My setup consists of Docker containers for running Pinot, Airflow, Debezium, and Trino. Like in the post on creating a warehouse with Doris, we'll create a `person` table in Postgres and replicate it into Kafka. We'll then ingest it into Pinot using its integrated Kafka consumer. Once that's done, we'll use Airflow to transform the data to create a view that makes it easier to work with it. Finally, we can use Superset to create reports. The intent of this post is to create a complete data platform that makes it possible to derive insights from data with minimal latency. The overall architecture looks like the following.  

{% asset_img Container.png %}

## Getting Started  

We'll begin by creating a schema for the `person` table in Pinot. This will then be used to create a realtime table. Since we want to use Pinot's upsert capability to maintain the latest record of each row, we'll ensure that we define the primary key correctly in the schema. In the case of the `person` table, it is the combination of the `id` and the `customer_id` field. The schema looks as follows.  

{% code lang:json %}
{
    "schemaName": "person",
    "dimensionFieldSpecs": [
        {
            "name": "id",
            "dataType": "LONG"

        },
        {
            "name": "customer_id",
            "dataType": "LONG"
        },
        {
            "name": "source",
            "dataType": "JSON"
        },
        {
            "name": "op",
            "dataType": "STRING"
        }
    ] ,
    "dateTimeFieldSpecs": [{
        "name": "ts_ms",
        "dataType": "LONG",
        "format" : "1:MILLISECONDS:EPOCH",
        "granularity": "1:MILLISECONDS"
    }],
    "primaryKeyColumns": [
        "id",
        "customer_id"
    ],
    "metricFieldSpecs": []
}
{% endcode %}  

We'll use the schema to create the realtime table in Pinot. Using `ingestionConfig` we'll extract fields out of the Debezium payload and into the columns defined above. This is defined below.

{% code lang:json %}
"ingestionConfig":{
      "transformConfigs":[
        {
            "columnName": "id",
            "transformFunction": "jsonPath(payload, '$.after.id')"
        },
        {
            "columnName": "customer_id",
            "transformFunction": "jsonPath(payload, '$.after.customer_id')"
        },
        {
           "columnName": "source",
           "transformFunction": "jsonPath(payload, '$.after')"
        },
        {
          "columnName": "op",
          "transformFunction": "jsonPath(payload, '$.op')"
        }
      ]
   }
{% endcode %}  

Next we'll create a table in Postgres to store the entries. The SQL query is given below.  

{% code lang:sql %}
CREATE TABLE person (
    id BIGSERIAL NOT NULL,
    customer_id BIGINT NOT NULL,
    name TEXT NOT NULL,
    PRIMARY KEY (id, customer_id)
);
{% endcode %}

Next we'll create a Debezium source connector to stream change data into Kafka.  

{% code lang:json %}
{
    "name": "person",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "database.hostname": "db",
        "database.user": "postgres",
        "database.password": "my-secret-pw",
        "database.dbname": "postgres",
        "database.server.name": "postgres",
        "table.include.list": ".*\\.person",
        "plugin.name": "pgoutput",
        "publication.autocreate.mode": "filtered",
        "time.precision.mode": "connect",
        "tombstones.on.delete": "false",
        "snapshot.mode": "initial",
        "heartbeat.interval.ms": "1000",
        "transforms": "route",
        "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
        "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
        "transforms.route.replacement": "$3",
        "event.processing.failure.handling.mode": "skip",
        "producer.override.compression.type": "snappy",
        "topic.prefix": "user_service"
    }
}
{% endcode %}

Finally, we'll use `curl` to send these configs to their appropriate endpoints, beginning with Debezium.  

{% code lang:bash %}
curl -H "Content-Type: application/json" -XPOST -d @debezium/person.json localhost:8083/connectors | jq .
{% endcode %}   

To create a table in Pinot we'll first create the schema followed by the table. The `curl` command is given below.  

{% code lang:bash %}
curl -F schemaName=@tables/001-person/person_schema.json localhost:9000/schemas | jq .
{% endcode %}

The command to create the table is given below.  

{% code lang:bash %}
curl -XPOST -H 'Content-Type: application/json' -d @tables/001-person/person_table.json localhost:9000/tables | jq .
{% endcode %}

With these steps done, the change data from Debezium will be ingested into Pinot. We can view this using Pinot's query console.

{% asset_img Pinot-1.png %}  

This is where we begin to integrate Airflow and Trino. While the data has been ingested into Pinot, we'll use Trino for querying. There are two main reasons for this. One, this allows usto federate queries across multiple sources. Two, Pinot's SQL capabilities are limited. For example, there is no support, as of writing, for creating views. To circumvent these we'll create a Hive connector in Trino and use it to query Pinot.  

The first step is to connect Trino and Pinot. We'll do this using the Pinot connector.  

{% code %}
CREATE CATALOG pinot USING pinot 
WITH (
    "pinot.controller-urls" = 'pinot-controller:9000'
);
{% endcode %}

Next we'll create the Hive connector. This will allow us to create views, and more importantly materialized views which act as intermediate datasets or final reports, which can be queried by Superset. I'm using AWS Glue instead of Hive so you'll have to change the configuration accordingly.

{% code %}
CREATE CATALOG hive USING hive
WITH (
    "hive.metastore" = 'glue',
    "hive.recursive-directories" = 'true',
    "hive.storage-format" = 'PARQUET',
    "hive.insert-existing-partitions-behavior" = 'APPEND',
    "fs.native-s3.enabled" = 'true',
    "s3.endpoint" = 'https://s3.us-east-1.amazonaws.com',
    "s3.region" = 'us-east-1',
    "s3.aws-access-key" = '...',
    "s3.aws-secret-key" = '...'
);
{% endcode %}

We'll create a schema to store the views and point it to an S3 bucket.

{% code %}
CREATE SCHEMA hive.views
WITH (
    "location" = 's3://your-bucket-name-here/views/'
);
{% endcode %}

We can then create a view on top of the Pinot table using Hive.  

{% code %}
CREATE OR REPLACE VIEW hive.views.person AS
SELECT id,
       customer_id,
       JSON_EXTRACT_SCALAR(source, '$.name') AS name,
       op
FROM pinot.default.person;
{% endcode %}

Finally, we'll query the view.

{% code %}
trino> SELECT * FROM hive.views.person;
 id | customer_id | name  | op
----+-------------+-------+----
  1 |           1 | Fasih | r
  2 |           2 | Alice | r
  3 |           3 | Bob   | r
(3 rows)
{% endcode %}

While this helps us ingest and query the data, we'll take this a step further and use Airflow to create the views instead. This allows us to create views which are time-constrained. For example, if we have an `order` table which contains all the orders placed by the customers, using Airflow allows to create views which are limited to, say, the last one year by adding a `WHERE` clause.   

We'll use the `TrinoOperator` that ships with Airflow and use it to create the view. To do this, we'll create an `sql` folder under the `dags` folder and place our query there. We'll then create the DAG and operator as follows.  

{% code lang:python %}
dag = DAG(
    dag_id="create_views",
    catchup=False,
    schedule="@daily",
    start_date=pendulum.now("GMT")
)

person = TrinoOperator(
    task_id="person",
    trino_conn_id="trino",
    sql="sql/views/person.sql",
    dag=dag
)
{% endcode %} 

## Workflow  

The kind of workflow this setup enables is the one where the data engineering team is responsible for ingesting the data into Pinot and creating the base views on top of it. The business intelligence / analytics engineering, and data science teams can then use Airflow to create datasets that they need. These can be created as materialized views to speed up reporting or training of machine learning models. Another advantage of this setup is that bringing in older data, say, of the last two years instead of one, is a matter of changing the query of the base view. This avoids complicated backfills and speeds things up significantly.  

As an aside, it is possible to use [DBT](https://www.getdbt.com/) instead of `TrinoOperator`. It can be used in conjunction with `TrinoOperator`, too. However, I preferred using the in-built operator to keep the stack simpler.

## Cost

Before we conclude, we'll quickly go over how to keep the cost of the Pinot cluster low while using this setup. [In the official documentation](https://docs.pinot.apache.org/operators/operating-pinot/separating-data-storage-by-age) it says that data can be seperated by age; older data can be stored in HDDs while the newer data can be stored in SSDs. This allows lowering the cost of the cluster.  

An alternative approach is to keep all the data in HDDs and load subsets into Hive for querying. This also allows changing the date range of the views by simply updating the queries. In essence, Pinot becomes the permanent storage for data while Trino and Hive become the intermediate query and storage layer.

That's it. That's how we can create a realtime data platform using Pinot, Trino, Debezium, and Airflow. 