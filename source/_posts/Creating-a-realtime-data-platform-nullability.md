---
title: Creating a realtime data platform - nullability
tags:
  - architecture
date: 2024-12-27 19:16:15
---

In the [previous post](/2024/12/26/Creating-a-realtime-data-platform-evolving-the-schema/) we looked at evolving the schema. We briefly discussed handling null values in columns when we added the `user_agent` column. It allows null values since `enableColumnBasedNullHandling` is set to true. However, we weren't able to see nullability in action since that column always had values in it. In this post we'll evolve the schema one more time and add columns that have null values in them. We'll see how to handle null values in Pinot queries, and how they differ from nulls in other databases. Let's dive right in.

## Getting started  

We'll begin by looking at the `source` payload that we've stored in Pinot.

{% code lang:json %}
{
    "user_id": 4,
    "cafe_id": 27,
    "address_id": 4,
    "created_at": 1735211553094,
    "id": 1,
    "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36 Edg/121.0.0.0",
    "status": 0
}
{% endcode %}  

In the payload above, we notice that there's `created_at` but no `updated_at` or `deleted_at`. That's because these have null values in the source table in Postgres. Let's update the schema and table definitions to store these fields.   

To update the schema, we'll add the following to `dateTimeFieldSpecs`.  

{% code lang:json %}
{
  "name": "updated_at",
  "dataType": "LONG",
  "format": "1:MILLISECONDS:EPOCH",
  "granularity": "1:MILLISECONDS",
  "defaultNullValue": "-1"
},
{
  "name": "deleted_at",
  "dataType": "LONG",
  "format": "1:MILLISECONDS:EPOCH",
  "granularity": "1:MILLISECONDS",
  "defaultNullValue": "-1"
}
{% endcode %}   

In the JSON above, we specify the usual fields just as we did for `created_at`. We also set `defaultNullValue`. This value will be used instead of null when these fields are extracted from the `source` payload. This is different from what you'd usually observe in a database that supports null values. The reason for this is that Pinot uses a [forward index](https://docs.pinot.apache.org/basics/indexing/forward-index) to store the values of each column. This index does not support storing null values and instead requires that a value be provided which will be stored in place of null. In our case, we've specified `-1`. The value that we specify as a default must be of the same data type as the the column. Since the two fields are of type `LONG`, specifying `-1` suffices.  

We'll PUT this schema using the following curl command.  

{% code %}
curl -XPUT -F schemaName=@tables/002-orders/orders_schema.json localhost:9000/schemas/orders | jq .
{% endcode %}

Next, we'll update the table definition by adding a couple of field-level transformations.  

{% code lang:json %}
{
    "columnName": "updated_at",
    "transformFunction": "jsonPath(source, '$.updated_at')"
},
{
    "columnName": "deleted_at",
    "transformFunction": "jsonPath(source, '$.deleted_at')"
}
{% endcode %}  

And POST it using the following curl command.   

{% code %}
curl -XPUT -H 'Content-Type: application/json' -d @tables/002-orders/orders_table.json localhost:9000/tables/orders | jq .
{% endcode %}  

Like we did last time, we'll reload all the segments using the following curl command.  

{% code %}
curl -XPOST localhost:9000/segments/orders/reload | jq .
{% endcode %}  

Now when we open the query console, we'll see the table with `updated_at` and `deleted_at` fields with their values set to `-1`.  

{% asset_img fields.png %}  

We know that we have a total of 5000 rows where the `deleted_at` field is set to null. This can be verified by running a count query in Pinot. This shows that although the values in the column are set to `-1`, Pinot identifies them as null and returns the correct result.

{% code lang:sql %}
SELECT COUNT(*)
FROM orders
WHERE deleted_at IS NULL;
{% endcode %}

{% asset_img count.png %}   

[A workaround suggested in the documentation](https://docs.pinot.apache.org/developers/advanced/null-value-support#appendix-workarounds-to-handle-null-values-without-storing-nulls) is to use comparison operators to compare against the value used in place of null. For example, the following query will produce the same result as the one shown above.   

{% code lang:sql %}
SELECT COUNT(*)
FROM orders
WHERE deleted_at = -1;
{% endcode %}  

{% asset_img alt.png %}  

That's it on how to handle null values in Pinot.