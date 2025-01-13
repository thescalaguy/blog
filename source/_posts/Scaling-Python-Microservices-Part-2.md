---
title: Scaling Python Microservices - sharding
tags:
  - architecture
date: 2024-05-12 17:21:14
---


In one of the [previous posts](/2024/03/25/Scaling-Python-Microservices/) we saw how we can scale a Python microservice and allow it to connect, theoretically, to an infinite number of databases. The way we did this is by fetching the database connection information at runtime from another microservice using the unique identifier of the customer. This allowed us to scale horizontally to some extent. However, there is still the limitation that the data of the customer may be so large that it would exceed the limit of the database when it is scaled vertically. In this post we'll look at how to extend the architecture we saw previously and shard the data across servers. We'll continue to use relational databases and see how we can shard the data using Postgres `PARTITION BY`.

## Before We Begin

The gist of scaling by sharding is to split the table into multiple partitions and let each of these be hosted on a separate host. For the purpose of this post we'll use a simple setup that consists of four partitions that are spread over two hosts. We'll use Postgres' Foreign Data Wrapper (FDW) to connect one instance of Postgres to another instance of Postgres. We'll store partitions in both these hosts, and create a table which uses these partitions. Querying this table would allow us to query data from all the partitions. 

## Getting Started

My setup has two instances of Postgres, both of which will host partitions. One of them will also contain the base table which will use these partitions. We'll begin by logging into the first instance and creating the FDW extension which ships natively with Postgres.  

{% code lang:sql %}
CREATE EXTENSION postgres_fdw;
{% endcode %}  

Next, we'll tell the first instance that there is a second instance of Postgres that we can connect to. Since both of these instances are running as Docker containers, I will use the hostname in the SQL query.  

{% code lang:sql %}
CREATE SERVER postgres_5 FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host 'postgres_5', dbname 'postgres');
{% endcode %}  

Next, we'll create a user mapping. This allows the user of the first instance to log into the second instance as one of its users. We're simply mapping the `postgres` user of the first instance to the `postgres` user of the second instance.

{% code lang:sql %}
CREATE USER MAPPING FOR postgres SERVER postgres_5 OPTIONS (user 'postgres', password 'my-secret-pw');
{% endcode %}

Next, we'll create the base table. There are a couple of things to notice. First, we use the `PARTITION BY` clause to specify that the table is partitioned. Second, there is no primary key on this table. Specifying a primary key prevents us from using foreign tables so we'll omit them.

{% code lang:sql %}
CREATE TABLE person (
  id BIGSERIAL NOT NULL,
  quarter BIGINT NOT NULL,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  customer_id TEXT NOT NULL
) PARTITION BY HASH (quarter);
{% endcode %}

Next, we'll create two partitions that reside on the first instance. We could, if the data were large enough, host each of these on separate instances. For the purpose of this post, we'll host them on the same instance. 

{% code lang:sql %}
CREATE TABLE person_0 PARTITION OF person FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE person_1 PARTITION OF person FOR VALUES WITH (MODULUS 4, REMAINDER 1);
{% endcode %}  

We'll now switch to the second instance and create two tables which will host the remaining two partitions.

{% code lang:sql %}
CREATE TABLE person_2 (
  id BIGSERIAL NOT NULL,
  quarter BIGINT NOT NULL,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  customer_id TEXT NOT NULL
);


CREATE TABLE person_3 (
  id BIGSERIAL NOT NULL,
  quarter BIGINT NOT NULL,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  customer_id TEXT NOT NULL
);
{% endcode %}

Once this is done, we'll go back to the first instance and designate these tables as partitions of the base table.

{% code lang:sql %}
CREATE FOREIGN TABLE person_2 PARTITION OF person FOR VALUES WITH (MODULUS 4, REMAINDER 2) SERVER postgres_5;
CREATE FOREIGN TABLE person_3 PARTITION OF person FOR VALUES WITH (MODULUS 4, REMAINDER 3) SERVER postgres_5;
{% endcode %}  

That's it. This is all we need to partition data across multiple Postgres hosts. We'll now run a benchmark to insert data into the table and its partitions.

{% code lang:bash %}
ab -p /dev/null -T "Content-Type: application/json" -n 5000 -c 100 -H "X-Customer-ID: 4" http://localhost:5000/person
{% endcode %}  

Once the benchmark is complete, we can query the base table to see that we have 5000 rows.  

{% code lang:sql %}
SELECT COUNT(*) FROM person;

count
5000
{% endcode %}  

What I like about this approach is that it is built using functionality that is native to Postgres - FDW, partitions, and external tables. Additionally, the sharding is transparent to the application; it sees a single Postgres instance.

Finito.
