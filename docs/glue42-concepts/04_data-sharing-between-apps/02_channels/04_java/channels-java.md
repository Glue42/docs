## Adding Channels to Your Window

To add the Channel Selector to your window, you need to enable the `channel` window option:

```java
glue.windows().register(handle, options -> options.channel())
```

## Creating Channel Context

A context data object can contain different types of data, e.g. `RIC` symbol, `ClientID`, `AccountID`:

```java
Map<String, Object> data = new LinkedHashMap<>();
data.put("RIC", "BMW.GR");
data.put("ClientID", 235399);
data.put("AccountID", "X2343");
```

You can create a Channel context object:

```java
ChannelContext<Map<String, Object>> channelContext = glue.channels().create(window);
```

## Subscribing for Data

When you want your application to track the current Channel data, you can use the `subscribe()` method of the Channel context object:

```java
channelContext.subscribe((ChannelContextDataSubscriber<Map<String, Object>>) (channel, data) -> {
    // Each time the Channel context data is updated, this method will be invoked.
});
```

## Publishing Data

To update the current Channel context data, use the `publish()` method:

```java
channelContext.publish(Collections.singletonMap("RIC", "VOD.L"));
```