## Overview

The Fidessa Tracking Groups are mapped via configuration to the [Glue42 Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) which allows passing context between Fidessa applications and Glue42 enabled applications.

Below is an example configuration of a Glue42 Channel which shows that the "Red" Glue42 Channel is mapped to Fidessa Tracking Group 5.

```json
{
    "name": "Red",
    "meta": {
        "color": "red",
        "fidessaGroup": {
            "index": 5,
            "readDataFieldPath": "",
            "writeDataFieldPath": ""
        }
    }
}
```

- `index` - the index of the Fidessa Tracking Group which to map to the respective Glue42 Channel;
- `readDataFieldPath` - specifies the field path to the Glue42 Channel data from where the Fidessa Connector reads the updated Channel value and then updates the Fidessa Tracking Group context.
- `writeDataFieldPath` - specifies the field path to the Glue42 Channel data where the Fidessa Connector writes the updated Fidessa Tracking Group context.

The mapping between the Glue42 Channels and the Fidessa Tracking Groups is preconfigured (in the `channels.json` file located in the `%localappdata%\Tick42\GlueDesktop\config` folder), but you can decide to assign different Fidessa Tracking Groups to different Glue42 Channels. For more information on configuring the Glue42 Channels, see the [Developers](../../../developers/configuration/channels/index.html) section.

## Context Shape

The context passed from the Fidessa Connector to the Glue42 enabled apps and vice versa has the following shape:

```typescript
{
    instrument: {
        // "VOD.L" or { fim: "VOD.L", bloomberg: "VOD:LN", ... }
        id: string | object;
        [prop: string] : any
    },
    client: string | object,
    order: {
        fidessaReference: string
    }
}
```

## Handling Channel Context

On how to update and subscribe for changes to a Channel context, see the [Glue42 Channels API](../../../glue42-concepts/data-sharing-between-apps/channels/javascript/index.html)

## Channel Updates

A Channel is updated every time the user selects an instrument, order or counterparty in a Fidessa application or in a Glue42 enabled application.

When an update from a Fidessa application arrives, the mapping service handles the conversion of instrument codes from FIM (Fidessa Instrument Mnemonic) to other standards (Bloomberg, Reuters Instrument Code, etc.). For instance, when a Fidessa application returns `"VOD.L"`, the connector translates that mnemonic to other standards as well and updates the context object accordingly:

```javascript
{
    instrument: {
        id: {
            bloomberg: "VOD:LN",
            fim: "VOD.L",
            ric: "VOD.L"
        }
    }
}
```

If there is no active mapping service, the value for the `instrument.id` property is the string literal received from Fidessa:

```javascript
{
    instrument: {
        id: "VOD.L"
    }
}
```

When a Glue42 enabled application updates the Channel context, the Fidessa Connector first tries to retrieve the instrument ID from the configured `fimPath` (see [Connection Settings](../connection/index.html#settings)) and if no instrument ID is available there, the value of the `instrument.id` property is sent to the mapping service. If no mapping service is available, or if the mapping service returns an object which doesn't contain an `instrument.id.fim` property, the update is ignored.