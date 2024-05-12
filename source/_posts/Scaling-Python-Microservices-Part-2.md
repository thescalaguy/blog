---
title: Scaling Python Microservices Part 2
tags:
  - architecture
date: 2024-05-12 17:21:14
---


In one of the [previous posts](/2024/03/25/Scaling-Python-Microservices/) we saw how we can scale a Python microservice and allow it to connect, theoretically, to an infinite number of databases. The way we did this is by fetching the database connection information at runtime from another microservice using the unique identifier of the customer. This allowed us to scale horizontally to some extent. However, there is still the limitation that the data of the customer may be so large that it would exceed the limit of the database when it is scaled vertically. In this post we'll look at how to extend the architecture we saw previously and shard the data across servers. We'll continue to use relational databases and see how we can shard the data using Postgres `PARTITION BY`.

## Before We Begin

The gist of scaling by sharding is to use [PgCat](https://github.com/postgresml/pgcat), a Postgres connection pooler with experimental support for sharding. Although I was able to get the data to shard across multple Docker containers, I was only able to read the data back from the first partition of the table which is in the first Docker container. The count query below, run against the PgCat instance, shows 2509 rows instead of 5000.

{% code lang:sql %}
postgres=# SELECT COUNT(*) FROM person;
 count 
-------
  2509
(1 row)
{% endcode %}

The statements to create one of the tables and its partition are given below.

{% code lang:sql %}
CREATE TABLE person (
  id BIGSERIAL NOT NULL,
  quarter BIGINT NOT NULL,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  customer_id TEXT NOT NULL,
  PRIMARY KEY (id, quarter)
) PARTITION BY HASH (quarter);

CREATE TABLE person_0 PARTITION OF person FOR VALUES WITH (MODULUS 4, REMAINDER 0);
{% endcode %}

## Getting Started

My setup consists of Docker containers for the API which will receive our request, another API which will return the connection information, an instance of PGCat, and four instances of Postgres which will store the sharded data. The API which returns the connection information returns the host and port of the PGCat instance rather than the actual database. PGCat then handles the reads and writes.

To use PGCat, we simply provide a configuration file which contains information on which database to connect to and how to shard the data.

{% code lang:yaml %}
pgcat:
  hostname: pgcat
  image: ghcr.io/postgresml/pgcat:latest
  ports:
    - "14432:6432"
    - "9930:9930"
  command:
    - "pgcat"
    - "/etc/pgcat/pgcat.toml"
  volumes:
    - "${PWD}/scripts/pgcat.toml:/etc/pgcat/pgcat.toml"
  depends_on:
    - postgres_4
{% endcode %}

We have the following information in the TOML file, along with the connection credentials.

{% code lang:toml %}
[pools.postgres]
sharding_function = "pg_bigint_hash"

# Automatically parse this from queries and route queries to the right shard!
automatic_sharding_key = "person.quarter"

# Shard 0
[pools.postgres.shards.0]
# [ host, port, role ]
servers = [
    [ "postgres_4", 5432, "primary" ],
    [ "postgres_4", 5432, "replica" ]
]
# Database name (e.g. "postgres")
database = "postgres"

[pools.postgres.shards.1]
# [ host, port, role ]
servers = [
    [ "postgres_5", 5432, "primary" ],
    [ "postgres_5", 5432, "replica" ]
]
# Database name (e.g. "postgres")
database = "postgres"

[pools.postgres.shards.2]
# [ host, port, role ]
servers = [
    [ "postgres_6", 5432, "primary" ],
    [ "postgres_6", 5432, "replica" ]
]
# Database name (e.g. "postgres")
database = "postgres"

[pools.postgres.shards.3]
# [ host, port, role ]
servers = [
    [ "postgres_7", 5432, "primary" ],
    [ "postgres_7", 5432, "replica" ]
]
# Database name (e.g. "postgres")
database = "postgres"
{% endcode %}

As you can see, we specify the hosts which will contain each of the shards. The `automatic_sharding_key` specifies the column which will be used to select the shard and is the fully-qualified name containing the table name and column name. Although the documentation states that the table name is optional, omitting the table name raises an error upon starting the PgCat container. I believe this is by design where one pool is meant to be used to shard a single table. If you have multiple tables you'll have to specify multiple pools in the configuration file which could perhaps be hosted on the same physical hosts.  

With the setup done, we can make requests to the API using Apache Bench.

{% code lang:bash %}
ab -p /dev/null -T "Content-Type: application/json" -n 5000 -c 100 -H "X-Customer-ID: 4" http://localhost:5000/person
{% endcode %}  

Since the connection information returned from the API points to the PgCat instance, the writes will be routed to the proper shards by PgCat. From the prespective of the application, we're connecting to a single Postgres instance.

That's it. That's how we can extend the architecture mentioned previously to now handle large customers with data that does not fit on a single machine.