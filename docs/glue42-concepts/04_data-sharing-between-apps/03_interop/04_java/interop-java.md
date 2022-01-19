## Method Registration

### Registering Methods

To offer a method to other applications, call `glue.interop().register()`, passing the method **definition** and a
callback to handle client invocations.

``` java
glue.interop().<Map<String, Object>, Map<String, Object>>register(
        MethodDefinition.builder("Sum").withSignature("int a, int b", "int answer").build(),
        (arg, caller) -> {
            int a = (Integer) arg.get("a");
            int b = (Integer) arg.get("b");
            return Collections.singletonMap("answer", a + b);
        });
```

Once a method is registered, it can be invoked from any Glue42 enabled app (web, native or Java).

### Method Definition

The method definition describes the Interop method your application is
offering. It has the following properties:

| Name| Description|
| ----------- | -------------- |
| name| **Required**. The name of the method, e.g. `OpenClientPerformance()`|
| displayName | The actual name of the method that should be used in UI applications, e.g. "Open Client Performance"|
| description | Description of what the method does, useful for documentation purposes and for UI clients, e.g. “Launches or activates the Client Performance application”|
| objectTypes | The entities this method is meant to operate on, e.g. `party`, `instrument`, `order`, etc. |

It is a good idea to specify `displayName` and `description` when defining a method. They can be used by a generic UI or by your own applications.

### Method Signature

The signature of a method is a comma-delimited string of parameters, defined as follows:

`type [array_modifier] [optional_modifier] parameter_name [composite_schema] [description]`

Where type is one of: `Bool`, `Int`, `Long`, `Double`, `String`, `DateTime`, `Composite` ("Composite" is explained below) and is case-insensitive, so `bool` and `BOOL` are the same thing.

Examples:

|Signature|Explanation|
|---------|-----------|
| `String pId, String? dynamicsId` | `pId` is required, `dynamicsId` is optional |
| `String branchCode, String[] gIds` | `branchCode` and `gIds` are required, `gIds` is an array of strings |
| `Composite: { String first, String last } name` | `name` is a composite parameter, and its schema is defined by 2 required strings - `first` and `last` |

`Composite` is a structure which contains one or more fields of type:

- **scalar** (bool, int, etc.)
- **array of scalars**
- a **composite** (nested structure)
- an **array of composites**

Using `Composite` you can define almost any non-recursive, non-self-referential structure.

### Asynchronous Results

You can obtain asynchronous results by using the `registerAsync()` method to register methods, which return a `Future` object:

``` java
glue.interop().registerAsync("getQuote", (arg, caller) -> {
    CompletableFuture<Map<String, Object>> future = doSomethingAsync(arg);
    return future;
});
```

The `doSomethingAsync()` method above returns a `Future` object.

## Method Invocation

### Invoking Methods

To invoke a method offered by other applications, call `glue.interop().invoke()`, passing the method **name** and **arguments**. Then use the returned `Future` to receive result or an error.

``` java
Map<String, Object> arg = new HashMap<>();
arg.put("a", 37);
arg.put("b", 5);
glue.interop().invoke("Sum", arg)
        .thenAccept(result ->
                            result.getReturned()
                                    .ifPresent(r -> System.out.println(r.get("answer"))))
        .toCompletableFuture().join();
```

## Multiple Responses

Invoking a method on multiple **Interop** instances produces **multiple**
responses.

This is how you can iterate over all responses:

``` java
result.forEach(r -> {
    if (r.getStatus().isSuccess())
    {
        System.out.printf("success:%s%n", r.getReturned().orElseGet(Collections::emptyMap));
    }
    else
    {
        System.out.printf("error:%s%n", r.getStatus().message().orElse(null));
    }
});
```

When the invocation result has multiple responses, calling `hasMultipleResponses()` on the result will return `true`.

## Discovery

### Discovering Methods

To list all available methods from all servers:

``` java
System.out.println(glue.interop().getMethods());
```

### Searching for Methods

#### Live Search Query

Glue42 Java offers a fluent API for finding Interop methods. It enables you to make a **live query** when you search for methods to invoke. This means that once you have an initial result from the search query, methods will be automatically added to/removed from that result when they become available or, respectively, unavailable.

The query starts from the `Search` class. You can use different methods of the `Search` class to specify criteria for the query:

```java
// here we are creating a Scheduler that we need for the example
// but you can also use an already created Scheduler
Scheduler scheduler = Scheduler.of(new Timer("search-scheduler", true));

Search
        .method(ReifiedType.OBJECT_MAP)     // return type of the method
        .name("Sum")                        // name of the method
        .localOnly(true)                    // offered only by local or by both remote and local servers
        .server(glue.interop().instance())  // specify a server offering the method (optional)
        .in(glue.interop(), scheduler, Duration.ofSeconds(10))
```

The `in()` method specifies the Interop instance (`glue.interop()`) which the query will use to track method added or removed events. The `scheduler` argument is of type `Scheduler` and is used to make the query asynchronous and non-blocking. The last parameter is a timeout for the search query.

Instead of `name()`, you can also use the `nameMatches()` method to pass a regex string or a `Pattern` for the name of the method(s) you are interested in.

#### Local and Remote Methods

You can look for methods offered both by local and remote servers by using the `localOnly()` method. Pass a boolean value to it to specify whether you need methods offered only by local servers or by both local and remote servers. The method `localOnly()` defaults to `true`, so if you omit it in the search, only methods offered by local servers will be returned.

Here is an example search for local methods:

```java
Search
        .method(ReifiedType.OBJECT_MAP)
        .name("Sum")
        .localOnly(true) //this can be omitted as it defaults to true
        .server(glue.interop().instance())
        .in(glue.interop(), scheduler, Duration.ofSeconds(10))
        .invoke(Collections.emptyMap(), // T args
                InvocationOptions
                        .builder()
                        .invocationTimeout(Duration.ofSeconds(10))
                        .invocationType(InvocationType.ASYNC)
                        .build())
        .whenComplete((result, error) -> {
            //logic for when the invocation completes
        });
```

### Discovering Servers

To list all servers offering methods:

``` java
System.out.println(glue.interop().getServers());
```

## Streaming

### Overview

Interop streams can be used by your application to:

- publish events which can be observed by other applications or provide real-time data (market data, news alerts, notifications, etc.) to other applications by publishing to an Interop stream;

- receive and react to the above events and data by creating an Interop stream subscription;

We call applications which create and publish to Interop streams *publishers*, and applications which subscribe to Interop streams - *subscribers*. An application can be both.

Interop Streams are used extensively in [**Glue42 Enterprise**](https://glue42.com/enterprise/) products and APIs.

<glue42 name="diagram" image="../../../../images/interop/interop-streaming.gif">

### Subscribing to a Stream

Subscribing to a stream is achieved by invoking `glue.interop().stream()`:

``` java
glue.interop()
        .stream("MarketData.LastTrades",
                Collections.singletonMap("symbol", "ORCL"))
        .thenAccept(stream -> stream.subscribe(new StreamSubscriber<Map<String, Object>>() {
            @Override
            public void onData(ServerMethod method, Map<String, Object> data) {
                // do something with the data
            }
        }));
```

### Closing a Stream Subscription

To close a stream subscription, invoke the `close()` or `closeAsync()` method on the subscription reference returned by the `subscribe()` method:

``` java
AsynchronousCloseable subscription =
        stream.subscribe(new StreamSubscriber<Map<String, Object>>() {});
subscription.closeAsync();
```

### Subscription Closed Notification

At any time, a stream subscription can be closed either because the publisher has shut down or due to an error.

``` java
stream.subscribe(new StreamSubscriber<Map<String, Object>>()
{
    @Override
    public void onSubscribe(StreamSubscription subscription)
    {
        subscription.onClose().thenRun(() -> {
            // called when the subscription is closed
        });
    }

    @Override
    public void onFailed(ServerMethod method, String reason)
    {
        // called if the subscription request is rejected
    }
});
```

### Publishing Data

To start publishing data, you need to register an Interop stream by
calling `glue.interop().register()` and providing a method definition and
stream subscription request handler.

``` java
glue.interop()
        .register(MethodDefinition.forName("Clock"),
                  StreamSubscriptionRequestHandler.accept())
        .thenAccept(stream -> {
            Map<String, Object> data =
                    Collections.singletonMap("CurrentTime", Instant.now().toEpochMilli());
            // will send data to all branches, as no branch is specified
            stream.send(data);
        });
```

`StreamSubscriptionRequestHandler.accept()` will accept all subscription requests on the default *branch* (see [Multicasting Data Using Branches](#streaming-multicasting_data_using_branches) below).


### Handling Subscription Requests

To control how your application accepts or rejects stream subscription requests, specify a custom handler. The handler receives a `StreamSubscriptionRequest` as a second argument and must return a `StreamConsumer` instance by invoking the `accept()`, `acceptOn()` or `reject()` methods of the `request`.

``` java
glue.interop()
        .<Map<String, Object>>register(
                MethodDefinition.builder("MarketData.LastTrades")
                        .withObjectType("Symbol")
                        .build(),
                request -> {
                    String app = request.getCaller().getApplication();
                    String symbol = (String) request.getArg().get("Symbol");

                    if (symbol != null)
                    {
                        System.out.printf("Accepting %s subscription on %s%n", app, symbol);
                        return request.acceptOn(symbol);
                    }
                    else
                    {
                        System.out.printf("Rejecting %s symbol not specified%n", app);
                        return request.reject("Symbol not specified");
                    }
                }
        );
```

### Handling New or Removed Consumers

To track when a consumer is added and/or removed, `StreamSubscriptionRequestHandler` provides the `onAdded()` and `onRemoved()` methods in which you can compose callbacks.

``` java
glue.interop()
        .register(
                MethodDefinition.builder("MarketData.LastTrades")
                        .withObjectType("Symbol")
                        .build(),
                StreamSubscriptionRequestHandler
                        .<Map<String, Object>>accept((arg, caller) -> (String) arg.get("Symbol"))
                        .onAdded(consumer -> {
                            // called when new consumer subscription is accepted
                        })
                        .onRemoved(consumer -> {
                            // called when a consumer subscription is removed
                        })
        );
```

In order to use these methods, you need an instance of type `StreamSubscriptionRequestHandler`. One way to achieve this, is to use one of the provided static factory methods instead of a lambda.

The following fragment creates a handler that is equivalent (if you ignore logging) with the one provided in the previous section:

``` java
StreamSubscriptionRequestHandler
        .<Map<String, Object>>accept((arg, caller) -> (String) arg.get("Symbol"))
```

The `StreamConsumer` reference can be used to:

- access the request parameters and the caller instance - `getArg()` and `getCaller()`;
- inspect the branch on which the consumer was accepted - `getBranch()`;
- push data directly to a consumer (unicast) - `send()`;
- close the subscription forcefully - `close()` or `closeAsync()`;

### Multicasting Data Using Branches

A single stream supports one or multiple named sub-streams that are called **branches**. In cases where it isn't necessary for a stream to be split into multiple sub-streams, a **default** branch is used.

``` java
glue.interop()
        .register(MethodDefinition.forName("Clock"),
                  StreamSubscriptionRequestHandler.accept())
        .thenAccept(stream -> {
            Map<String, Object> data =
                    Collections.singletonMap("CurrentTime", Instant.now().toEpochMilli());
            // data will be sent to branches with names "Consumers" and "Providers"
            stream.send(data, "Consumers", "Providers");
        });
```

The `stream` reference is of type `StreamProducer` and can be used to:

- send data to specific stream branch(es). If no branch is specified, that data is sent to all branches:

```java
stream.send(data, "Consumers", "Providers");
```

- list all available branches:

```java
Map<String, StreamBranch> allBranches = stream.getBranches();
```

- list all stream consumers, regardless of the branch they are on:

```java
List<StreamConsumer<?>> allConsumers = stream.getConsumers();
```

If your stream publishing code uses **branches** (e.g., creates a branch for each unique set of subscription arguments and associates consumers with that branch), whenever a data arrives from your underlying source, you can use that branch to publish data instead of manually iterating over all consumers to send data to the interested clients.

```java
String branchKey = "Providers";
StreamBranch branch = stream.getBranches().get(branchKey);

if (branch != null) {
    branch.push(data);
    System.out.printf("Pushed data to all subscribers on branch \"%s\"!%n", branch.getKey());
} else {
    System.out.printf("Branch \"%s\" doesn't exist!%n", branchKey);
}
```

To close all subscriptions on this branch, call `closeAsync()` on the `StreamBranch` instance:

```java
branch.closeAsync();
```

### Stream Discovery

Streams are special Interop methods on which `isSupportsStreaming()` returns `true`. You can use the [Interop Discovery](#discovery) to find available streams.