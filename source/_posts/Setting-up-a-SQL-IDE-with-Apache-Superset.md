---
title: Setting up a SQL IDE with Apache Superset
tags:
  - misc
date: 2024-02-08 20:01:39
---

I have a few requirements when choosing a SQL IDE. I'd like to use something that is relatively modern, has good support for a wide variety of databases, and allows making charts and graphs. I looked at Metabase, TablePlus, DBeaver, DataGrip, and Apache Superset. All of these tools have a feature or two that is missing in the other. For example, DataGrip allows creating entity relationship diagrams that make it easy to understand how the tables are related to each other; many of the tools do not have this feature. The free version of TablePlus allows creating only one set of charts; Metabase and Superset allow creating many of these. For my day-to-day requirements, as you may have guessed from the title of this post, Apache Superset is the best fit. This post shows how to quickly set up Superset using Docker Compose.  

## Why Superset  

One of the things I often need to do is to monitor the performance of a system. For example, building upon the previous post where I talked about a hypothetical notification delivery system, counting how many notifications are being sent out per day, both as an aggregate metric across channels, and per-channel. This can be tracked by generating StatsD metrics, and charting them in Grafana. However, chances are that we'd need to display them to the customer, and also to other non-technical stakeholders. To do this we'd need our systems to generate records in the database, and have these be stored in a data warehouse that we can eventually query. This is where Superset is useful; it has built-in IDE that allows exploratory data analysis, and the ability to create charts once we've finalised the SQL query. The query can then be made part of the application serving analytics, or the BI tool that is used to by the stakeholders. 

Another reason is that it is an open-source, actively-developed project. This means bug fixes and improvements will be shipped at a decent cadence; we have a tool that will stay modern as long as we keep updating the local setup.  

Finally, because it is an open-source project, it has good documentation, and a helpful community. Both of these make using a technology a pleasant experience.

## Setting up Superset  

Superset supports a wide vartiety of databases. However, we need to install drivers within the Docker containers to support additional databases. For the sake of this example, we'll install the Snowflake driver. To set up Superset locally we need to clone the git repo. Let's start by doing that. 

{% code lang:bash %}
git clone https://github.com/apache/superset.git
{% endcode %}  

Next we navigate to the repo and add the Snowflake driver.

{% code lang:bash %}
cd superset
echo "snowflake-sqlalchemy" >> ./docker/requirements-local.txt
echo "cryptography==39.0.1" >> ./docker/requirements-local.txt
{% endcode %}  

I had to add the `cryptography` package manually because the first time I set up Superset, logs showed that database migrations did not run becasuse it was missing. 

Next we checkout the latest stable version of the repo. As of writing, it is 3.0.1.  

{% code lang:bash %}
git checkout 3.0.1
{% endcode %}  

Now we bring up the containers.  

{% code lang:bash %}
TAG=3.0.1 docker-compose -f docker-compose-non-dev.yml up -d
{% endcode %}  

This will start all the containers we need for version 3.0.1 of Superset. Note that if we add newer drivers, we'll need to rebuild the images. Navigate to the [Superset login page](http://localhost:8088) and use admin as both the username and password. We can now follow the documentation on how to set up a data source, and use the SQL IDE to write queries.  

That's it. We've set up Superset.