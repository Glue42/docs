## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

The Pub/Sub API is accessible through `glue.Bus`.

## Publish

### To All Applications

To publish a message on a specific topic to all subscribed applications, use the `Publish()` method. It accepts the message topic and the data to publish as arguments:

```csharp
var topic = "stocks";
var data = new { RIC = "AAPL.O" };

await glue.Bus.Publish(topic, data);
```

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-all.gif">

### To Specific Applications

Use the message options builder as a third argument of `Publish()` to make an application publish a message only to specific applications that have subscribed to a topic.

The example below demonstrates how to publish a message to another application (or to multiple instances of it) with a specific name:

```csharp
var topic = "stocks";
var data = new { RIC = "AAPL.O" };
var appName = "app-name";

await glue.Bus.Publish(topic, data, (messageOptions) =>
{
    messageOptions.WithTargetOptions(target => target.WithApplicationName(appName));
}
```

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-specific-app.gif">

The Pub/Sub API compares the `appName` argument with the identity of each application subscribed to the topic and delivers the message only to subscribers with a matching application name.

The example below demonstrates how to publish messages with a specific routing key:

```csharp
var topic = "stocks";
var data = new { RIC = "AAPL.O" };
var routingKey = "portfolio";

await glue.Bus.Publish(topic, data, options => options.WithRoutingKey(routingKey));
```

The Pub/Sub API delivers messages with a routing key to all subscribers with the same routing key and to the ones with no routing key.

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-routing.gif">

## Subscribe

### Messages from Any Application

To subscribe for messages from all applications on a specific topic, use the `Subscribe()` method. Upon successful subscription, it returns a subscription object of type `IDisposable`. Use its `Dispose()` method to stop receiving messages on that topic.

Provide the topic on which you want to receive messages and a handler function for the messages:

```csharp
var topic = "stocks";

IDisposable stocksSubscription = await glue.Bus.Subscribe(topic, (message) =>
{
    Console.WriteLine($"Received data: {message.Data}");
});

// Closing the subscription.
stocksSubscription.Dispose();
```

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-all.gif">

### Messages from Specific Applications

<!-- Use the message options builder as a third argument of `Subscribe()` to make an application subscribe for messages only from specific applications:

```csharp
var topic = "stocks";
var target = new Dictionary<string, object>{{ "application" , "app-name"}};

await glue.Bus.Subscribe(topic, (message) =>
{
    Console.WriteLine($"Received data: {message.Data}");
},
options => options.WithTarget(target));
```

The Pub/Sub API compares the `target` argument with the identity of the publisher. It invokes the message handler only if the `target` argument entries match the respective entries in the identity of the publisher.

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-specific-app.gif"> -->

The example below demonstrates how to subscribe for messages with a specific routing key:

```csharp
var topic = "stocks";
var routingKey = "portfolio";

await glue.Bus.Subscribe(topic, (message) =>
{
    Console.WriteLine($"Received data: {message.Data}");
},
options => options.WithRoutingKey(routingKey));
```

The Pub/Sub API invokes the handler only for messages with a matching routing key and for the ones with no routing key.

<glue42 name="diagram" image="../../../../images/pub-sub/pub-sub-routing.gif">

## Typed Message Buses

The .NET Pub/Sub API allows you to create typed message buses if you want to ensure that the publisher or the subscriber pushes or receives specific type of data.

To create a typed message bus, use the `GetTypedBus()` method:

```csharp
// Push or receive messages containing only integer data.
var intBus = glue.Bus.GetTypedbus<int>();

// Push or receive messages containing only data of type `AppState`.
var appStateBus = glue.Bus.GetTypesBus<AppState>();
```

### Publish

To publish messages on a typed bus:

```csharp
var appStateBus = glue.Bus.GetTypesBus<AppState>();
var topic = "app-state";
var data =  new AppState { DarkThemeOn = true };

await appStateBus.Publish(topic, data);
```

### Subscribe

To subscribe for messages on a typed bus:

```csharp
var intBus = glue.Bus.GetTypedbus<int>();
var topic = "quantity";

await intBus.Subscribe(topic, (message) =>
{
    Console.WriteLine($"Received data: {message.Data}");
});
```

Or:

```csharp
var topic = "quantity";

await glue.Bus.Subscribe<int>(topic, (message) =>
{
    Console.WriteLine($"Received data: {message.Data}");
});
```

### Untyped Data

The data which which fails to deserialize to the required type is accessible through the `UntypedData` property of the received message. Use the `HasDeserializationError` boolean flag to determine whether a deserialization error has occurred. Use the `WithIgnoreDeserializationErrors()` method of the message options builder to specify whether the message handler should be invoked with the untyped data:

```csharp
await intBus.Subscribe(topic, (message) =>
{
    if (message.HasDeserializationError)
    {
        Console.WriteLine($"Received wrong data type: {message.UntypedData}");
    }
    else
    {
        Console.WriteLine($"Received correct data type: {message.Data}")
    }
},
options => options.WithIgnoreDeserializationErrors(true));
```