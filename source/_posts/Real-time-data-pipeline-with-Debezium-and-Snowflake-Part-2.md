---
title: Real-time data pipeline with Debezium and Snowflake Part 2
tags:
  - data
date: 2022-06-12 12:36:34
---

In the previous post we saw the overall architecture of the pipeline that we are going to build. In this post we will actually build it using Docker. By the end of this post we will have Kafka, MySQL, and Kafka Connect running as containers. We will query the Kafka Connect REST API to to get some basic information back before we run connectors.  

## Getting Started  

Start by opening a terminal and creating a directory called `connect`.  

{% code %}
mkdir connect
{% endcode %} 

Then create the `Dockerfile` and `docker-compose.yml` files. We will populate these later.  

{% code %}
cd connect
touch Dockerfile
touch docker-compose.yml
{% endcode %}  

### Dockerfile  

We will start with the `Dockerfile`. In it we will extend the Confluent Kafka Connect base image and install two connectors in it. This image comes with all the things necessary to run Kafka Connect and the `confluent-hub` utility to install new connectors. It's just three lines long so go ahead and paste the following in your file.  

{% code %}
FROM confluentinc/cp-kafka-connect-base:6.2.4
RUN confluent-hub install --no-prompt debezium/debezium-connector-mysql:1.8.1
RUN confluent-hub install --no-prompt snowflakeinc/snowflake-kafka-connector:1.7.2
{% endcode %}   

We are installing two connectors here. The first is the Debezium MySQL source connector which will read MySQL and write the binlogs to Kafka, and the other is the Snowflake sink connector which will read from Kafka and write data to Snowflake.  

### docker-compose.yml  

In the `docker-compose.yml` file we will build the image from the `Dockerfile` we just created and also add services for Zookeeper, Kafka, and MySQL. Go ahead and add the following code snippet to your compose file. 

{% code lang:yaml %}
version: '3'
services:
  zookeeper:
    hostname: zookeeper
    image: debezium/zookeeper:1.6
    ports:
      - 2181:2181
      - 2888:2888
      - 3888:3888
  kafka:
    hostname: kafka
    image: debezium/kafka:1.6
    ports:
      - 9092:9092
    links:
      - zookeeper
    environment:
      - ZOOKEEPER_CONNECT=zookeeper:2181
      - CREATE_TOPICS=kafka_connect_config:1:1:compact,kafka_connect_offsets:1:1:compact,kafka_connect_status:1:1:compact
    depends_on:
      - zookeeper
  mysql:
    hostname: mysql
    image: debezium/example-mysql:1.6
    ports:
      - 3306:3306
    environment:
      - MYSQL_USER=mysql
      - MYSQL_ROOT_PASSWORD=my-secret-pw
      - MYSQL_PASSWORD=my-secret-pw
  connect:
    hostname: connect
    build: .
    image: connect:latest
    ports:
      - 8083:8083
    depends_on:
      - kafka
      - mysql
      - zookeeper
    environment:
      - CONNECT_BOOTSTRAP_SERVERS=kafka:9092
      - CONNECT_CONFIG_STORAGE_TOPIC=kafka_connect_config
      - CONNECT_OFFSET_STORAGE_TOPIC=kafka_connect_offsets
      - CONNECT_STATUS_STORAGE_TOPIC=kafka_connect_status
      - CONNECT_GROUP_ID=kafka_connect
      - CONNECT_KEY_CONVERTER=org.apache.kafka.connect.json.JsonConverter
      - CONNECT_VALUE_CONVERTER=org.apache.kafka.connect.json.JsonConverter
      - CONNECT_REST_ADVERTISED_HOST_NAME=localhost
      - CONNECT_PLUGIN_PATH=/usr/share/java,/usr/share/confluent-hub-components
{% endcode %}  

We are specifying a Kafka service and creating a few topics by adding them in the `CREATE_TOPICS` environment variable. These are topics that will be used by Kafka Connect when it starts running.   

Kafka Connect needs some configuration. The `CONNECT_CONFIG_STORAGE_TOPIC` stores the configuration information for the connectors we will run. The `CONNECT_OFFSET_STORAGE_TOPIC` topic stores the offsets. `CONNECT_STATUS_STORAGE_TOPIC` stores the state of the connectors. Finally, `CONNECT_PLUGIN_PATH` specifies the directories from which to load the connectors. We are exposing port 8083 because that is where we will make REST API calls. 

Now we have everything we need. We'll go ahead and start the services.  

## Running the services  

Let's run the services.  

{% code %}
docker-compose up --force-recreate
{% endcode %}  

Once the services are up we can verify that they are running by using `curl`.  

{% code %}
curl http://localhost:8083
{% endcode %}  

This will return a JSON reponse similar to the following  

{% code lang:javascript %}
{
  "version": "6.2.4-ccs",
  "commit": "2a9ff1dfd68974a1",
  "kafka_cluster_id": "c4xWNl-4ReGHedvaW8YgPQ"
}
{% endcode %}  

Next we will see which connector plugins are available for us to run. This is done by issuing the following `curl` command:  

{% code %}
curl http://localhost:8083/connector-plugins
{% endcode %}  

This returns the following JSON response:  

{% code lang:javascript %}
[
  {
    "class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
    "type": "sink",
    "version": "1.7.2"
  },
  {
    "class": "io.debezium.connector.mysql.MySqlConnector",
    "type": "source",
    "version": "1.8.1.Final"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
    "type": "source",
    "version": "1"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
    "type": "source",
    "version": "1"
  },
  {
    "class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
    "type": "source",
    "version": "1"
  }
]
{% endcode %}  

We can see that the Snowflake sink connector and MySQL source connector are available for us to run.  

## Conclusion  

In this blog post we took the first step towards building a real-time data pipeline with Debezium and Snowflake. We put together the infrastructure which helps us run Kafka Connect. We made some basic `curl` requests to see that Kafka Connect is running properly.  

In the next post we will create the Snowflake account and set it up so that we can run connectors to ingest data.  

Finito.