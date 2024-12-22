---
title: Creating a realtime data platform - creating the data
tags:
  - architecture
date: 2024-12-22 12:33:43
---


In the [previous post](/2024/12/18/Creating-a-data-wrehouse-with-Apache-Pinot-and-Debezium/) we saw the overall design of the platform. We saw how the components of the system are divided into three separate categories: those that bring the data in, those that create datasets on this data, and those that display visualizations. Starting from this post, we're going to start building the system. We'll work with the data of a fictitious online cafe that we'll populate using a Python script. In subsequent posts, we'll ingest this data into the platform and create visualizations on top of it.

## Before we begin  

The setup, for this post, consists of a Docker container for Postgres which is a part of the compose file. We'll bring up the container before we begin populating the database.

## The data  

We'll create and populate a table which stores the orders placed by the customer. The table contains, among other fields, the id of the user who placed the order, the id of the address where the order needs to be delivered, the status of the order, and the user agent of the device used to place the order. The code snippet below shows how the model is represented as a Python class.

{% code lang:python %}
class Order(peewee.Model):
    """An order placed by the customer."""

    class Meta:
        table_name = "orders"
        database = database

    id = peewee.BigAutoField()
    user_id = peewee.IntegerField()
    address_id = peewee.IntegerField()
    cafe_id = peewee.IntegerField()
    partner_id = peewee.IntegerField(null=True)
    created_at = peewee.DateTimeField(default=datetime.datetime.now)
    updated_at = peewee.DateTimeField(null=True)
    deleted_at = peewee.DateTimeField(null=True)
    status = peewee.IntegerField(default=0)
    user_agent = peewee.TextField()
{% endcode %}  

Once we've created this class, we'll write a function which creates instances of this class and persists them in the database. There are classes representing the cafe, the addresses saved by the user, and the delivery partner who will be assigned to deliver the order. However, these have been left out for the sake of brevity. The code snippet below shows this function.  

{% code lang:python %}
def create_orders(
    users: list[User],
    addresses: list[Address],
    cafes: list[Cafe],
    partners: list[Partner],
    n: int = 100,
) -> list[Order]:
    ua = UserAgent()
    orders = []

    def base_order() -> dict:
        cafe = cafes[random.randint(0, len(cafes) - 1)]
        user = users[random.randint(0, len(users) - 1)]
        addr = [_ for _ in addresses if _.user_id == user.id][0]
        user_agent = ua.random

        return {
            "user_id": user.id,
            "address_id": addr.id,
            "cafe_id": cafe.id,
            "user_agent": user_agent,
        }

    for _ in range(n):
        data = {**base_order(), "status": OrderStatus.PLACED.value}
        order = Order.create(**data)
        orders.append(order)

    return orders
{% endcode %}  

Once we have all our classes and functions in place, we'll run the script which populates the data.

{% code %}
python faker/data.py
{% endcode %}  

We can now query the database to see our data.  

{% asset_img data.png %}  

This is it for the second part of the series.