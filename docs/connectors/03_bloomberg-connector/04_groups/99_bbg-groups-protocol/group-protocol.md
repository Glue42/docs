## Overview

The Bloomberg Groups are visual color-coded named groups (much like [Glue42 Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html)) which the user can select from the UI of Bloomberg Components. They provide data synchronization across Bloomberg Components (e.g., if two components are in the same colored group, when an instrument changes in one component, the other component is updated respectively). The Bloomberg Connector API provides configurable mapping between the Bloomberg Groups and the Glue42 Channels.

Below you can see how the Bloomberg Connector automatically synchronizes the Glue42 enabled demo applications that are on the channel mapped to the respective Bloomberg group of the running Bloomberg components:

![BBG-Channels](../../../../images/bloomberg/bbg-channels.gif)

*Note that the Glue42 Channels are mapped via configuration to the default names of the Bloomberg Groups. Therefore, modifying the default name of a group in the Bloomberg Terminal will cause the mapping between the Bloomberg Groups and the Glue42 Channels to break. Any change you make to a Bloomberg Group name you must reflect in the `channles.json` file (located in `%LocalAppData%\Tick42\GlueDesktop\config`) which contains the configuration for the Glue42 Channels.*

## Groups Operations

### Get Group Names

Get the known groups as a list of strings with the group names:

- Interop method name: `"T42.BBG.GetGroups"`
- Accepts: `void`
- Returns: Array of strings with the available Bloomberg group names;

Example:

```javascript
const result = await glue.interop.invoke("T42.BBG.GetGroups");

// Example value: ["Group-B", "Group-A"]
const groups = result.returned.Result;

if (groups && groups.length > 0) {
    // Use the Bloomberg groups here.
};
```

### Get Group Context

Get the context of a group by name:

- Interop method name: `"T42.BBG.GetGroupContext"`
- Accepts: The group name as a string;
- Returns: The context of the group as a string;

Example:
```javascript
const invocationOptions = { group: "Group-A" };
const result = await glue.interop.invoke("T42.BBG.GetGroupContext", invocationOptions);

// Example value: "VOD LN Equity"
const context = result.returned.Result;
```

### Update Group Context

Update the context of a group by name:

```csharp
(void) T42.BBG.SetGroupContext (string group, string context)
```

- Interop method name: `"T42.BBG.SetGroupContext"`
- Accepts: The name of the group as a string and context as a string with which to update the group;
- Returns: `void`

Example:
```javascript
const invocationOptions = {
    group: "Group-A",
    context: "BARC LN Equity"
};

glue.interop.invoke("T42.BBG.SetGroupContext", invocationOptions);
```

## Streaming Events

A Glue42 enabled app can subscribe for and be notified about group changes:

- Interop method name: `"T42.BBG.GroupsChanged"`
- Accepts: `void`
- Returns: An array of objects with `name` and `value` string properties, representing the name of the changed group and the value of its new context;

Example:

```javascript
const subscription = await glue.interop.subscribe("T42.BBG.GroupsChanged");

subscription.onData(handleGroupChanges);

function handleGroupChanges(streamData) {
    // Example value: [{ name: "Group-A", value: "VOD LN Equity" }]
    const changes = streamData.data.groups;
    // Handle group changes here.
};
```

## Channels Synchronization

Currently, [**Glue42 Enterprise**](https://glue42.com/enterprise/) has Channel definitions for all Bloomberg Groups. The Glue42 [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) are defined in `%LocalAppData%\Tick42\GlueDesktop\config\channels.json`.

The binding configuration for the Bloomberg Groups to the Glue42 Channels can be found in the `meta` property of the Glue42 Channel definition:

```json
{
    "blpGroup": {
        "name" : "Group-G",
        "readDataFieldPath" : "data.partyPortfolio.ric",
        "writeDataFieldPath" : "data.partyPortfolio.ric",
        "read" : "ric",
        "write": "ric"
    }
}
```

- `name` - specifies the Bloomberg Group to bind to the respective Glue42 Channel;

- `readDataFieldPath` - specifies the field path to the Glue42 Channel data from where the Bloomberg Connector reads the updated Channel value and then updates the Bloomberg Group context.

- `writeDataFieldPath` - specifies the field path to the Glue42 Channel data, where the Bloomberg Connector writes the updated Bloomberg Group context.

- `read` and `write` - specify whether the type of the context to read/write is `RIC` or `BLP`;