## Adding Channels to Your App

To add the Channel Selector to your window, set `"allowChannels"` to `true` in your app configuration file under the `"details"` top-level key:

```json
{
    "title": "Client List 🔗",
    "type": "window",
    "name": "channelsclientlist",
    "icon": "https://dev-enterprise-demos.tick42.com/resources/icons/clients.ico",
    "details": {
        "url": "https://dev-enterprise-demos.tick42.com/client-list-portfolio-contact/#/clientlist",
        "mode": "tab",
        "allowChannels": true
    }
}
```

In some cases, you may want to show the Channel Selector, but prevent the user from interacting with it. Set the `"readOnlyChannelSelector"` property to `true` under the `"details"` top-level key in your app configuration file to achieve this:

```json
{
    "title": "Client List 🔗",
    "type": "window",
    "name": "channelsclientlist",
    "icon": "https://dev-enterprise-demos.tick42.com/resources/icons/clients.ico",
    "details": {
        "url": "https://dev-enterprise-demos.tick42.com/client-list-portfolio-contact/#/clientlist",
        "mode": "tab",
        "allowChannels": true,
        "readOnlyChannelSelector": true
    }
}
```

You can also set the Channel Selector as read-only on a global level from the `system.json` file under the `"windows"` top-level key:

```json
{
    "windows": {
        "readOnlyChannelSelector": true
    }
}
```

*Note that the settings in your app configuration file will override the global settings in the `system.json` file.*

The Channels API is disabled by default. To enable it, set the `channels` property of the configuration object to `true` when initializing the Glue42 library:

```javascript
const config = { channels: true };

window.glue = await Glue(config);
```

*See the JavaScript [Channels example](https://github.com/Glue42/js-examples/tree/master/channels) on GitHub.*

## Current Channel

The Channels API is accessible through the [`glue.channels`](../../../../reference/glue/latest/channels/index.html) object.

To get the name of the Channel your app is currently on, use the [`my()`](../../../../reference/glue/latest/channels/index.html#API-my) method:

```javascript
const myChannel = glue.channels.my();
```

## All Channels

To get a list of all Channel names, use the [`all()`](../../../../reference/glue/latest/channels/index.html#API-all) method:

```javascript
const channelNames = await glue.channels.all();
```

## Joining or Leaving a Channel

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

To make your app join a Channel programmatically, use the [`join()`](../../../../reference/glue/latest/channels/index.html#API-join) method and specify the name of the Channel to join:

```javascript
await glue.channels.join("Red");
```

To leave the Channel your app is currently on, use the [`leave()`](../../../../reference/glue/latest/channels/index.html#API-leave) method:

```javascript
await glue.channels.leave();
```

## Retrieving Channel Context

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

To get the context of a Channel, use the [`get()`](../../../../reference/glue/latest/channels/index.html#API-get) method which accepts a Channel name as a required parameter:

```javascript
const data = await glue.channels.get("Green");
```

To get a list of the contexts of all Channels, use the [`list()`](../../../../reference/glue/latest/channels/index.html#API-list) method:

```javascript
const channelContexts = await glue.channels.list();
```

## Subscribing for Data

To track the data in the current Channel, use the [`subscribe()`](../../../../reference/glue/latest/channels/index.html#API-subscribe) method:

```javascript
const handler = (data) => {
    // The callback will be invoked each time the data is updated.
    console.log(data);
};

// Subscribe for updates from the Channel your app is currently on.
glue.channels.subscribe(handler);
```

The callback receives the data from the Channel and information about the current Channel.

The callback will be invoked in three cases:
- the `data` property of the Channel you are currently on is updated;
- the user has switched the Channel and you are receiving a snapshot of the new Channel data;
- your app isn't joined to a Channel anymore (e.g., the user has deselected the current Channel). In this case, both `data` and `channelInfo` will be `undefined`;

To subscribe for updates from a specific Channel, use the [`subscribeFor()`](../../../../reference/glue/latest/channels/index.html#API-subscribeFor) method:

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

```javascript
const channelName = "Green";
const handler = (data) => {
    // The callback will be invoked each time the data is updated.
    console.log(data);
};

await glue.channels.subscribeFor(channelName, handler);
```

The `subscribeFor()` method accepts a Channel name as a first parameter and a callback to handle Channel data updates.

Use the unsubscribe function returned by `subscribe()` and `subscribeFor()` to stop tracking updates of the Channel data:

```javascript
const unsubscribe = await glue.channels.subscribeFor(channelName, handler);

unsubscribe();
```

The handlers passed to the `subscribe()` and `subscribeFor()` methods also accept the Channel context and the updating Interop [Instance](../../../../reference/glue/latest/interop/index.html#Instance) peer ID as second and third arguments. The [ChannelContext](../../../../reference/glue/latest/channels/index.html#ChannelContext) object contains the name of the Channel and the Channel meta data and the updating Interop instance peer ID can be used to identify the app updating the Channel:

```javascript
const handler = (data, channelContext, updaterID) => {
    // Check the current Interop instance peer ID against the updating instance ID.
    const isUpdatedByMe = glue.interop.instance.peerId === updaterID;

    if(!isUpdatedByMe) {
        // Another app has published in the Channel.
        console.log(`App "${updaterID}" has published "${JSON.stringify(data)}" in Channel "${channelContext.name}".`);
    };
};

glue.channels.subscribe(handler);
```

## Publishing Data

To update the context of the Channel, use [`publish()`](../../../../reference/glue/latest/channels/index.html#API-publish). The `publish()` method accepts two parameters - data to publish (required) and an optional Channel ID specifying which Channel context to update. If you don't specify a Channel ID, the current Channel will be updated.

Updating the current Channel:

```javascript
const data = { RIC: "VOD.L" };

await glue.channels.publish(data);
```

Updating a specific Channel:

```javascript
const data = { RIC: "VOD.L" };
const channelName = "Green";

await glue.channels.publish(data, channelName);
```

Note that a Channel may contain multiple data structures, e.g. `RIC` and `clientId`. When executing the code above, only the `RIC` field will be updated, leaving the other fields of the context unchanged.

The [`publish()`](../../../../reference/glue/latest/channels/index.html#API-publish) method will throw an exception if you aren't on a Channel and try to publish data.

## Channel Events

If you want to monitor how your app moves between Channels, subscribe for updates with the [`onChanged()`](../../../../reference/glue/latest/channels/index.html#API-onChanged) method:

```javascript
const handler = (newChannel) => {
    if (newChannel) {
        // Handle the case where you have switched to another Channel.
        console.log(newChannel);
    } else {
        // Handle the case where your app isn't joined to any Channel
        // (e.g., the user has deselected the current Channel).
        console.log("No Channel selected.")
    };
};

glue.channels.onChanged(handler);
```

## Reference

For a complete list of the available Channels API methods and properties, see the [Channels API Reference Documentation](../../../../reference/glue/latest/channels/index.html).