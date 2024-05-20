---
title: Scaling Python Microservices Part 3
tags:
  - architecture
date: 2024-05-19 17:35:46
---


In a [previous blog post](/2024/03/21/Setting-up-a-Python-microservice/) we'd seen how to use OpenTelemetry and Parca to instrument two microservcies. Imagine that our architecture has grown and we now have many microservices. For example, there is a microservice that is used to send emails, another which keeps track of user profile information, etc. We'd previously hardcoded the HTTP address of the second microservice to make a call from the first microservice. We could have, in a production environment, added a DNS entry which points to the load balancer and used that instead. However, when there are many microservcies, it becomes cumbersome to add these entries. Furthermore, any new microservice which depends on another microservice now needs to have the address as a part of its configuration. Maintaining a single repository, which can be used to find the microservices and their addresses, becomes difficult to maintain. An easier way to find services is to use service discovery - a dynamic way for one service to find another. In this post we'll modify the previous architecture and add service discovery using Zookeeper.  

## Before We Begin  

In a nutshell, we'll use Zookeeper for service registration and discovery. We'll modify our `create_app` factory function to make one more function call which registers the service as it starts. The function call creates an ephemeral node in Zookeeper at a specified path and stores the host and port. Any service which needs to find another service looks for it on the same path. We'll create a small library for all of this which will make things easier.


## Getting Started  

We'll begin by creating a simple attr class to represent a service.  

{% code lang:python %}
@define(frozen=True)
class Service:
    host: str
    port: int
{% endcode %}  

We'll be using the `kazoo` Python library to interact with Zookeeper. The next step is to create a private variable which will be used to store the Zookeeper client.  

{% code lang:python %}
_zk: KazooClient | None = None
{% endcode %}  

Next we'll add the function which will be called as a part of the `create_app` factory function.  

{% code lang:python %}
def init_zookeeper_and_register():
    _init_zookeeper()
    _register_service()
{% endcode %}  

In this function we'll first initialize the connection to Zookeeper and then register our service. We'll see both of these functions next, starting with the one to init Zookeeper.  

{% code lang:python %}
def _init_zookeeper():
    global _zk

    if not _zk:
        _zk = KazooClient(hosts='127.0.0.1:2181')
        _zk.start(timeout=5)
        _zk.add_listener(_state_change_listener)
{% endcode %}  


The `_zk` variable is meant to be a singleton. I was unable to find whether this is thread-safe or not but for the sake of this post we'll assume it is. The `start` method creates a connection to Zookeeper synchronously and raises an error if no connection could be made in `timeout` seconds. We also add a listener to respond to changes in the state of a Zookeeper connection. This enables us to respond to scenarios where the connection drops momentarily. Since we'll be creating ephemeral nodes in Zookeeper, they'll be removed in the case of a session loss and would need to be recreated. In the listener we recreate the node upon a successful reconnection.  

Next we'll look at the function to register the service.  

{% code lang:python %}
def _register_service():
    global _zk

    service = os.environ.get("SERVICE")
    host = socket.getfqdn()
    port = os.environ.get("PORT")

    assert service
    assert host
    assert port

    identifier = str(uuid4())
    path = f"/providers/{service}/{identifier}"
    data = {"host": host, "port": int(port)}
    data_bytes = json.dumps(data).encode("utf-8")

    _zk.create(path, data_bytes, ephemeral=True, makepath=True)
{% endcode %}  

We get the name of the service and the port it is running on as environment variables. The host is retrieved as the fully-qualified domain name of the machine we're on. Although the example is running on my local machine, it may work on a cloud provider like AWS. We then create a UUID identifier for the service, and a path on which the service will be registered. The information stored on the path is a JSON containing the host and the port of the service. Finally, we create an ephemeral node on the path and store the host and port.  

Next we'll look at the function which handles changes in connection state.  

{% code lang:python %}
def _state_change_listener(state):
    if state == KazooState.CONNECTED:
        _register_service()
{% endcode %}  

We simply re-register the service upon reconnection.  

Finally, we'll look at the function which retrieves an instance of the service from Zookeeper.  

{% code lang:python %}
def get_service_from_zookeeper(service: str) -> Service | None:
    global _zk
    assert _zk

    path = f"/providers/{service}"
    children = _zk.get_children(path)

    if not children:
        return None

    idx = random.randint(0, len(children) - 1)

    child = children[idx]
    config, _ = _zk.get(f"/providers/{service}/{child}")
    config = config.decode("utf-8")
    config = json.loads(config)

    return Service(**config)
{% endcode %}  

To get a service we get all the children at the path `/providers/SERVICE_NAME`. This returns a list of UUIDs, each of which represents an instance of the service. We randomly pick one of these instances and fetch the information associated with it. What we get back is a two-tuple containing the host and port, and an instance of ZStat which we can ignore. We decode and parse the returned information to get an instance of `dict`. This is then used to return an instance of `Service`. We can then use this information to make a call to the returned service.  

This is all we need to create a small set of utility functions to register and discover services. All we need to do to register the service as a part of the Flask application process is to add a function call in the `create_app` factory function.  

{% code lang:python %}
def create_app() -> Flask:
    ...

    # -- Initialize Zookeeper and register the service.
    init_zookeeper_and_register()

    return app

{% endcode %}  

Finally, we'll retrieve the information about the second service in the first service right before we make the call.  

{% code lang:python %}
def make_request() -> dict:
    service = get_service_from_zookeeper(service="second")
    url = f"http://{service.host}:{service.port}"
    ...
{% endcode %}  

As an aside, the service will automatically be deregistered once it stops running. This is because we created an ephemeral node in Zookeeper and it only persists as long as the session that created it is alive. When the service stops, the session is disconnected, and the node is removed from Zookeeper.

### Rationale  

The rationale behind registering and discovering services from Zookeeper is to make it easy to create new services and to find the ones that we need. It's even more convenient when there is a shared library which contains the code that we saw above. For the sake of simplicity, we'll assume all our services are written in Python and there is a single library that we need. The library could also contain an enum that represents all the services that are registered with Zookeeper. For example, to make a call to an email microservice, we could have code that looks like the following.  

{% code lang:python %}
email_service = Service.EMAIL_SERVICE.value
service = get_service_from_zookeeper(service=email_service)
{% endcode %}  

This, in my opinion, is much simpler than adding a DNS entry. The benefits add up over time and result in code that is both readable and maintainable.  

That's it. That's how we can register and retrieve services from Zookeeper. [Code is available on Github.](https://github.com/thescalaguy/microservice)