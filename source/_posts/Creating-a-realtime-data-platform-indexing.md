---
title: Creating a realtime data platform - indexing
tags:
  - architecture
date: 2025-01-12 21:31:29
---

In the [previous post](/2025/01/10/Creating-a-realtime-data-platform-embedding/) we saw how we can embed a Superset dashboard in a webpage using a Flask app. In this post we'll look at creating indexes on the tables that are stored in Pinot. We'll look at the various types of indexes that can be created and then create indexes of our own to speed up the queries.  

## Getting started  

Very briefly, a database index is a data structure that improves retrieving data from the tables. It is used to quickly find the rows that we're interested in without having to scan the entire table. Pinot supports different types of indexes that can be used to speed up different types of queries. Knowing the access patterns of the queries helps us decide which indexes we'd like to create. We'll quickly look at each type of index that Pinot provides, write a few queries, and then create indexes to speed them up. Let's start by looking at the various index types that are available in Pinot.  

Pinot supports the following types of indexes -- bloom filter index, forward index, FST index, geospatial index, inverted index, JSON index, range index, star-tree index, text search index, and timestamp index. In the sections that follow, we'll look at how to create an inverted index, and a range index to speed up our queries. Let's start with the inverted index.  

Let's say we'd like to find out the sum of the amounts of orders placed on January 1st. One way to write this query would be to convert the `created_at` time to a date on the fly. This would require us to look at every row we have in the table and compare it against the date we're looking for. Another way to write this query would be to convert `created_at` to a date during ingestion time and create an inverted index on it. Creating an inverted index would store a mapping of each date to the document IDs with the same date. This will enable us to filter only those documents which have the date we're looking for instead of having to look at each one of them.

We'd have to update our table and schema for the `orders` table. We'll start by adding a column to our schema definition which will store the date as a string.  

{% code lang:json %}
{
  "name": "date",
  "dataType": "STRING"
}
{% endcode %}

Next, we'll convert the `created_at` time to a date and store it in the `date` column. To do this, we'll create a field-level transformation in the table definition.   

{% code lang:json %}
{
  "columnName": "date",
  "transformFunction": "toDateTime(created_at, 'yyyy-MM-dd')"
}
{% endcode %}

Next, we'll enable the inverted index on the `date` column by adding the following to the table definition.

{% code lang:json %}
{
  "tableIndexConfig": {
    "invertedIndexColumns": [
      "date"
    ],
    "createInvertedIndexDuringSegmentGeneration": true
}
{% endcode %}

Once we PUT these configs, we'll reload the segments so that the index is created. We can now write our query in the Pinot console.  

{% code lang:sql %}
SELECT "date", SUM(amount) as total
FROM orders
WHERE "date" = '2024-01-01'
GROUP BY 1; 
{% endcode %} 

We can verify that the index was used by looking at the explain plan for the query.  

{% code lang:sql %}
EXPLAIN PLAN FOR
SELECT "date", SUM(amount) as total
FROM orders
WHERE "date" = '2024-01-01'
GROUP BY 1;
{% endcode %}  

This produces the following result.  

{% asset_img screen_1.png %}  

Looking at the explain plan, we find the following line. It tells us that the index was used to look up the date '2024-01-01'

{% code %}
FILTER_SORTED_INDEX(indexLookUp:sorted_index,operator:EQ,predicate:date = '2024-01-01')
{% endcode %}

Let's modify the query slightly and look for the daily total for the last 90 days. It looks as follows.  

{% code lang:sql %}
SELECT "date", SUM(amount) as total
FROM orders
WHERE created_at >= ago('P90D')
GROUP BY 1
ORDER BY 1
{% endcode %}

The `ago()` function takes as argument a duration string and returns milliseconds since epoch. We compare it against `created_at`, which is also expressed as milliseconds since epoch, to get the final result. Let's look at the explain plan for the query.  

{% asset_img screen_2.png %}  

We find the following line which indicates that an index was not used.  

{% code %}
FILTER_FULL_SCAN(operator:RANGE,predicate:created_at >= '1728886039203')
{% endcode %}

We can improve the performance of the query by using a range index which allows us to efficiently query over a range of values. This helps us speed up queries which involve comparison operators like less than, greater than, etc. To create a range index, we'll have to update the table definition and specify the column on which we'd like to create the index. We'll add the following to the table definition.  

{% code lang:json %}
{
  "tableIndexConfig": {
    "rangeIndexColumns": [
      "created_at"
    ]
}
{% endcode %}

Like we did previously, we'll PUT this config and reload the segments. We'll once again look at the explain plan for the query above. 

{% asset_img screen_3.png %}

We now find the following line which indicates that the range index was used.  

{% code %}
FILTER_RANGE_INDEX(indexLookUp:range_index,operator:RANGE,predicate:created_at >= '1728887647851')
{% endcode %}

This was a quick overview of using indexes to speed up queries on Pinot. We looked at the inverted index and the range index. In the next post, we'll look at the star-tree index which lets us compute pre-aggregations on columns and speeds up the result of operations like SUM and COUNT. We'll also look at stream processing to generate new events from the ones emitted by Debezium.
