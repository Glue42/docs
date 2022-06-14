## Overview

With the Glue42 Bloomberg Connector you can bring various Bloomberg services to your Glue42 enabled applications. The Bloomberg Connector is a .NET application which is part of [**Glue42 Enterprise**](https://glue42.com/enterprise/) and is auto started with [**Glue42 Enterprise**](https://glue42.com/enterprise/) as a hidden application. It offers sets of protocols based on [Interop methods](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html) for connecting to the [Bloomberg Market Data service](https://www.bloomberg.com/professional/product/market-data/), as well as to the [Bloomberg Terminal](https://www.bloomberg.com/professional/solution/bloomberg-terminal/). Currently, a more convenient JavaScript (TypeScript) wrapping API is offered only for connecting to the Market Data service. With the provided API protocols, however, you can access the Bloomberg Market Data service and control the Bloomberg Terminal features with any JavaScript, .NET, Java or COM technology.

The Bloomberg Connector also offers a Bloomberg Simulator where the API can be used without having an actual Bloomberg Terminal installed.

## Available Bloomberg Functionalities

The Market Data service and several other functionalities of the Bloomberg Terminal become available to you through the Bloomberg Connector API.

### Accessing BBG Functionalities Programmatically

All available Bloomberg functionalities are accessible through the methods registered by the Bloomberg Connector. We have provided complete descriptions of the protocols for using these functionalities (method descriptions and concise explanations of what they accept and return). Currently, only the Market Data service has a JavaScript (TypeScript) wrapper. To use all other Bloomberg functionalities, you have to invoke the respective methods from your Glue42 enabled apps (see how to invoke [Interop](../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html) methods through the Glue42 [JavaScript](../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html#method_invocation), [.NET](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html#imperative_model-method_invocation), [Java](../../../glue42-concepts/data-sharing-between-apps/interop/java/index.html#method_invocation) and [VBA](../../../glue42-concepts/data-sharing-between-apps/interop/vba/index.html#method_invocation) APIs).

Below is a sample representation of a protocol method description followed by an invocation example:

- Interop method name: `"T42.BBG.GetGroups"`
- Accepts: `void`
- Returns: Array of strings with the available Bloomberg group names;

Example:

```javascript
const result = await glue.interop.invoke("T42.BBG.GetGroups");
const groups = result.returned.Result;

if (groups && groups.length > 0) {
    // Use the Bloomberg groups here.
};
```

### Market Data

The Bloomberg Market Data offers retrieval of real-time/delayed streaming market data and static reference market data for securities through subscription and request/response mechanisms. Using the available [Market Data protocols](../market-data/bbg-mdf-protocol/index.html) or [APIs](../market-data/javascript/index.html) you can easily acquire market data provided by Bloomberg in order to show it, use it or redistribute it throughout your applications. The following functionalities are available:

- [Subscription to streaming market data](../market-data/javascript/index.html#request_types-market_data_subscription);
- [Historical Data requests](../market-data/javascript/index.html#request_types-historical_data);
- [Reference Data requests](../market-data/javascript/index.html#request_types-reference_data);
- [Instrument List requests](../market-data/javascript/index.html#request_types-instrument_list);
- [Intraday Bar requests](../market-data/javascript/index.html#request_types-intraday_bar);
- [Snapshot requests](../market-data/javascript/index.html#request_types-snapshot);
- [Field List requests](../market-data/javascript/index.html#request_types-field_list);
- [Field Search requests](../market-data/javascript/index.html#request_types-field_search);
- [User Entitlements requests](../market-data/javascript/index.html#request_types-user_entitlements);

### Bloomberg Groups

The Bloomberg Groups are visual color-coded named groups (much like [Glue42 Channels](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html)) which the user can select from the UI of Bloomberg Components. They provide data synchronization across Bloomberg Components (e.g., if two components are in the same colored group, when an instrument changes in one component, the other component is updated respectively). The Bloomberg Connector API provides configurable mapping between the Bloomberg Groups and the Glue42 Channels. The following events and functionalities are available:

- BBG Group context changed event (e.g., instrument update);
- BBG Group context update via a Glue42 invocation method;
- BBG Group context read (get) via a Glue42 invocation method;
- BBG Group context update via a [Glue42 Channel](../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html);
- Keeping a Glue42 Channel synchronized with a BBG Group context;

### Bloomberg Worksheets

The Bloomberg Worksheets are spreadsheets containing a list of securities to monitor. The following Bloomberg Worksheets manipulations are available:

- Get a BBG Worksheets list via a Glue42 invocation method;
- Get the BBG Worksheet context (instrument list) via a Glue42 invocation method;
- Create BBG worksheets;
- Update (add/remove/set) BBG Worksheet context via a Glue42 invocation method;

### Bloomberg Components

The Bloomberg Components are Bloomberg applications (windows) providing Bloomberg Terminal functionalities. The following Bloomberg Components manipulations are available:

- Host Bloomberg Components (hosted in WPF windows, can be created on demand) in Glue42 windows that can participate in Glue42 layouts and workspaces;
- Get/set context properties of the Bloomberg components via a Glue42 invocation method;

### Bloomberg Functions

The Bloomberg Terminal has built-in functions which execute various kinds of data analyses on markets or securities. The Glue42 Bloomberg Connector API allows you to invoke Bloomberg functions with a custom set of arguments.