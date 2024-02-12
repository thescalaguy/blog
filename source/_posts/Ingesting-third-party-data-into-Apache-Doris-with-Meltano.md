---
title: Ingesting third-party data into Apache Doris with Meltano
tags:
  - architecture
date: 2024-02-12 11:15:48
---


A common requirement for a data warehouse is to store data from third-party sources like SaaS tools. For example, results of surveys conducted by Product or Marketing teams, or status of JIRA tickets. While most of these tools provide an option to export data, it could be very limited. For example, they'd only allow downloading it as a CSV file from their dashboard as a one-off download. Some of them provide APIs to export data which is a lot more convenient. Of those that do provide a direct integration with a warehouse, it'd be with the commercial ones like Snowflake. If we're using a different warehouse, it is going to take some engineering effort to periodically get this data.   

In this post we'll look at how we can streamline this ingestion. As always, we'll use open-source technologies to do this. Like in my previous post, we'll use Doris as our data warehouse. We'll use the open-source tool Meltano to fetch data from a Github repository. While this is not going to fully replace the need to write custom code, it will help us benefit from the collection of open-source connectors that come with Meltano.  

## Setting things up  

Like in my previous post, my setup consists of Docker containers for running Doris locally. There's a Postgres container that runs as a part of this setup. We'll install Meltano, and ingest data into a Postgres database. We'll then create a catalog in Doris that points to the data ingested in Postgres. While it may be possible to ingest the data directly into Doris by letting Meltano write it to S3, I find it much easier to write it to a database.   

## Getting started  

For the purpose of this post we'll will be modifying the example on Meltano's website to ingest data from Github to Postgres. Meltano is available as a pip package and I already have a Conda environment in which it is installed. I'll gloss over the steps that show how to create a Meltano project for the sake of keeping the post brief, and jump right into the ones where we add the Github extractor, the Postgres loader, and configure them. 

Let's start by adding the loader, and the extractor plugins.  

{% code lang:bash %}
meltano add extractor tap-github
meltano add loader target-postgres
{% endcode %}  

We'll then configure them one-by-one, starting with the Github plugin. We'll add the repository we'd like to fetch data from, and the auth token that will be used to authenticate with Github.  

{% code lang:bash %}
meltano config tap-github set repositories '["thescalaguy/blog"]'
meltano config tap-github set auth_token '...'
{% endcode %}  

Next we'll configure the Postgres plugin. We'll ingest the data into a schema called "github" in the "ingest" database.

{% code lang:bash %}
meltano config target-postgres set user postgres
meltano config target-postgres set password my-secret-pw
meltano config target-postgres set database ingest
meltano config target-postgres set port 6432
meltano config target-postgres set host 127.0.0.1
meltano config target-postgres set default_target_schema github
{% endcode %}  

Finally, we run the pipeline.  

{% code lang:bash %}
meltano run tap-github target-postgres
{% endcode %}  

A successful run of the pipeline will create a whole bunch of tables in Postgres. One such table is "repositories", and we'll query it next.  

{% code lang:sql %}
SELECT id, repo, org, name FROM repositories;
{% endcode %}  

The query above gives us the following row.

{% code %}
| id       | repo | org         | name |
|----------|------|-------------|------|
| 95359149 | blog | thescalaguy | blog |
{% endcode %}

We can now set the pipeline to run on a schedule by slightly modifying the example given in [Meltano's "Getting Started" guide](https://docs.meltano.com/getting-started#schedule-pipelines-to-run-regularly). 

{% code lang:bash %}
meltano schedule add github-to-postgres --extractor tap-github --loader target-postgres --interval @daily
{% endcode %}  

We'll now create a catalog in Doris that points to the database in Postgres. To do this we need to execute the `CREATE CATALOG` command. Notice that we've specified the `driver_url` that points to Maven. This is a lot easier than manually downloading the JAR, and making it available in the Docker container.

{% code lang:sql %}
CREATE CATALOG meltano PROPERTIES (
    "type"="jdbc",
    "user"="postgres",
    "password"="my-secret-pw",
    "jdbc_url" = "jdbc:postgresql://192.168.0.107:6432/ingest",
    "driver_url" = "https://repo1.maven.org/maven2/org/postgresql/postgresql/42.7.1/postgresql-42.7.1.jar",
    "driver_class" = "org.postgresql.Driver"
);
{% endcode %}  

We have one last step before we can query the data, and that is to fix a few symlinks in the containers. It looks like the code is looking for CA certificates at a location different than where they actually are. From what I could glean from the documentation, perhaps the base image has changed and the paths need to be upadted. In any case, here's how to fix it both in the FE and BE containers by `docker exec`-ing into them.  

{% code lang:bash %}
root@be:/# mkdir -p /etc/pki/tls/certs/
root@be:/# ln -s /etc/ssl/certs/ca-certificates.crt /etc/pki/tls/certs/ca-bundle.crt
{% endcode %}  

Now we'll run a `SELECT` query. Notice how we're writing the `FROM` clause; it is `catalog.schema.table`.

{% code lang:sql %}
SELECT id, repo, org, name 
FROM meltano.github.repositories;
{% endcode %}

There are, however, limitations to what we can read from the Postgres table. Some of the column types are not supported. Changing the query to `SELECT *` will raise an error that "topics" column is not supported. We can describe the table in the catalog to check which of the columns cannot be read. 

{% code lang:sql %}
DESC meltano.github.repositories;
{% endcode %}  

The result shows "UNSUPPORTED_TYPE" next to "topics" column. This is perhaps because Doris uses MySQL protocol and there's no equivalent mapping bewteen Postgres and MySQL for that type. One way to mitigate this is to create a new column using Meltano's `stream_maps`, and parse it in Doris.  

That's it. That's how we can ingest data from third-party systems into Doris.