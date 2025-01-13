---
title: Scaling Python Microservices - dynamic databases
tags:
  - architecture
date: 2024-03-25 12:39:42
---


In one of the [previous posts](2024/03/21/Setting-up-a-Python-microservice/) we saw how to set up a Python microservice. This post is a continuation and shows how to scale it. Specifically, we'll look at handling larger volumes of data by sharding it across databases. We'll shard based on the customer (or tenant, or user, etc.) making the request, and route it to the right database. While load tests on my local machine show promise, the pattern outlined is far from production-ready. 

## Before We Begin

We'll assume that we'd like to create a service that can handle growing volume of data. To accommodate this we'd like to shard the data across databases based on the user making the request. For the sake of simplicity we'll assume that the customer is a user of a SaaS platform. The customers are heterogenous in their volume of data — some small, some large, some so large that they require their own dedicated database. 

We'll develop a library, and a sharding microservice. Everything outlined in this post is very specific to the Python ecosystem and the libraries chosen but I am hopeful that the ideas can be translated to a language of your choice. We'll use Flask and Peewee to do the routing and create a pattern that allows transitioning from a single database to multiple databases. 

The setup is fully Dockerized and consists of three Postgres databases, a sharding service, and an api service.

## Getting Started 

In a nutshell, we'd like to look at a header in the request and decide which database to connect to. This needs to happen as soon as the request is received. Flask makes this easy by allowing us to execute functions before and after receiving a request and we'll leverage them to do the routing. 

### Library

We'll begin by creating a library that allows connecting to the right database.

{% code lang:python %}
@attr.s(auto_attribs=True, frozen=True)
class Shard:
    config: APIConfig | StandaloneConfig
    identifier_field: str

    def db(self, identifier: Identifier) -> Database:
        credentials = self._db_credentials(identifier=identifier)

        credentials_dict = attr.asdict(credentials)  # noqa
        credentials_dict.pop("flavor")

        if credentials.flavor == DatabaseFlavor.POSTGRES:
            return PostgresqlDatabase(**credentials_dict)

        if credentials.flavor == DatabaseFlavor.MYSQL:
            return MySQLDatabase(**credentials_dict)

    def _db_credentials(self, identifier: Identifier) -> DBCredentials:
        if isinstance(self.config, StandaloneConfig):
            credentials = attr.asdict(self.config)  # noqa
            return DBCredentials(**credentials)
        return self._fetch_credentials_from_api(identifier=identifier)

    def _fetch_credentials_from_api(self, identifier: Identifier) -> DBCredentials:
        url = f"{self.config.endpoint}/write/{str(identifier)}"
        response = requests.get(url)
        response.raise_for_status()
        json = response.json()
        return cattrs.structure(json, DBCredentials)
{% endcode %}  

An instance of `Shard` is responsible for connecting to the right database depending on how it is configured. In "standalone" mode, it connects to a single database for all customers. This is helpful when creating the microservice for the first time. In "api" mode it makes a request to the sharding microservice. This is helpful when we'd like to scale the service. The API returns the credentials for the appropriate database depending on the identifier passed to it. The `identifier_field` is a column which must be present in all tables. For example, every table must have a "customer_id" column.  

We'll add a helper function to create an instance of the `Shard`. This makes it easy to transition from standalone to api mode by simply setting a few environment variables.

{% code lang:python %}
def from_env_variables() -> Shard:
    mode = os.environ.get(EnvironmentVariables.SHARDING_MODE)
    identifier_field = os.environ.get(EnvironmentVariables.SHARDING_IDENTIFIER_FIELD)

    if mode == ShardingMode.API:
        endpoint = os.environ.get(EnvironmentVariables.SHARDING_API_ENDPOINT)
        config = APIConfig(endpoint=endpoint)
        return Shard(config=config, identifier_field=identifier_field)

    if mode == ShardingMode.STANDALONE:
        host = os.environ.get(EnvironmentVariables.SHARDING_HOST)
        port = os.environ.get(EnvironmentVariables.SHARDING_PORT)
        user = os.environ.get(EnvironmentVariables.SHARDING_USER)
        password = os.environ.get(EnvironmentVariables.SHARDING_PASSWORD)
        database = os.environ.get(EnvironmentVariables.SHARDING_DATABASE)
        flavor = os.environ.get(EnvironmentVariables.SHARDING_FLAVOR)

        config = StandaloneConfig(
            host=host,
            port=int(port),
            user=user,
            password=password,
            database=database,
            flavor=flavor,
        )

        return Shard(config=config, identifier_field=identifier_field)
{% endcode %}

### API service

We'll add request hooks to the API service which will use an instance of the `Shard` to connect to the right database.  

{% code lang:python %}
_shard = sharding.from_env_variables()


def _before_request():
    identifier = request.headers.get("X-Customer-ID")
    g.db = _shard.db(identifier=identifier)


def _after_request(response):
    g.db.close()
    return response


api = Blueprint("api", __name__)
api.before_app_request(_before_request)
api.after_app_request(_after_request)
{% endcode %}  

We're creating an instance of `Shard` and using it to retrieve the appropriate database. This is then stored in the per-request global `g`. This lets us use the same database throughout the context of the request. Finally, we register the before and after hooks.  

We'll now add functions to the library which allow saving and retrieving the data using the database stored in `g`.  

{% code lang:python %}
def save_with_db(
    instance: Model,
    db: Database,
    force_insert: bool = False,
):
    identifier_field = os.environ.get(EnvironmentVariables.SHARDING_IDENTIFIER_FIELD)
    identifier = getattr(instance, identifier_field)
    assert identifier, "identifier field is not set on the instance"

    with db.bind_ctx(models=[instance.__class__]):
        instance.save(force_insert=force_insert)
{% endcode %}  

What allows us to switch the database at runtime is the `bind_ctx` method of the Peewee `Database` instance. This temporarily binds the model to the database that was retrieved using the `Shard`. In essence, we're storing and retrieving the data from the right database.  

Next we'll add a simple Peewee model that represents a person.  

{% code lang:python %}
class Person(Model):
    class Meta:
        model_metadata_class = ThreadSafeDatabaseMetadata
    
    id = BigAutoField(primary_key=True)
    name = TextField()
    address = TextField()
    customer_id = TextField()
{% endcode %}  

We'll add an endpoint which will let us save a row with some randomly generated data.  

{% code lang:python %}
@api.post("/person/<string:customer_id>")
def post_person(customer_id: str) -> dict:
    fake = Faker()
    person = Person(
        name=fake.name(),
        address=fake.address(),
        customer_id=customer_id,
    )
    save_with_db(instance=person, db=g.db, force_insert=True)
    return {"success": True}
{% endcode %}  

### Sharding service 

We'll add an endpoint to the sharding service which will return the database to connect to over an API.  

{% code lang:python %}
@api.get("/write/<string:identifier>")
def get_write_db(identifier: str):
    return {
        "host": f"postgres_{identifier}",
        "port": 5432,
        "flavor": "postgres",
        "user": "postgres",
        "password": "my-secret-pw",
        "database": "postgres",
    }
{% endcode %}

Here we're selecting the right database depending on the customer making the request. For the sake of this demo the information is mostly static but in a production scenario this would come from a meta database that the sharding service connects to.  

### Databases  

We'll now create tables in each of the three databases.  

{% code lang:sql %}
CREATE TABLE person (
  id BIGSERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  address TEXT NOT NULL,
  customer_id TEXT NOT NULL
);
{% endcode %}  

### Testing  

We'll add a small script which uses Apache Bench to send requests for three different customers.  

{% code lang:bash %}
#!/bin/sh

ab -p /dev/null -T "Content-Type: application/json" -n 1000 -c 100 -H "X-Customer-ID: 1" http://localhost:5000/person/1 &
ab -p /dev/null -T "Content-Type: application/json" -n 1000 -c 100 -H "X-Customer-ID: 2" http://localhost:5000/person/2 &
ab -p /dev/null -T "Content-Type: application/json" -n 1000 -c 100 -H "X-Customer-ID: 3" http://localhost:5000/person/3 &
{% endcode %}

We'll run the script and wait for it to complete.  

{% code lang:bash %}
./bench.sh
{% endcode %} 

We'll now connect to one of the databases and check the data. I'm connecting to the third instance of Postgres which should have data for the customer with ID "3". We'll first check the count of the rows to see that all 1000 rows are present, and then check the customer ID to ensure that requests are properly routed in a multithreaded environment.  

{% code lang:sql %}
SELECT COUNT(*) FROM person;
{% endcode %}


This returns the following:

{% code %}
count
1000
{% endcode %}

We'll now check for the customer ID stored in the database.

{% code lang:sql %}
SELECT DISTINCT customer_id FROM person;
{% endcode %}

This returns the following:

{% code %}
customer_id
3
{% endcode %}  

## Conclusion  

That's it. That's how we can connect a Flask service to multiple databases dynamically.
