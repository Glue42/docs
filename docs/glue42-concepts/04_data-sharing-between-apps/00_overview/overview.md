## Overview

[**Glue42 Enterprise**](https://glue42.com/enterprise/) provides a wide variety of Interop services – [Request/Response](../interop/overview/index.html), [Streaming](../interop/javascript/index.html#streaming), [Shared Contexts](../shared-contexts/overview/index.html), [Channels](../channels/overview/index.html). The rich Interop functionality of [**Glue42 Enterprise**](https://glue42.com/enterprise/) offers you a wide choice of solution options when developing your apps. Each Interop service is best applied in certain scenarios:

## Request/Response

If you have an app with existing functionalities which you want to offer to other applications within [**Glue42 Enterprise**](https://glue42.com/enterprise/), you can use the Request/Response Interop API. Your application can register an Interop method which invokes the respective functionality of your app. Other Glue42 enabled applications can discover this Interop method and invoke it in order to use the said functionality.

## Streaming 

If you need to stream or capture real-time data, you can use special Interop methods for publishing or subscribing to data streams – just like with Request/Response, an app can register a method for publishing a stream and another app can invoke that method to subscribe for the stream and receive the published data.

## Shared Contexts 

If you need to synchronize data across multiple applications (e.g., the user clicks on a client name in a client list and you want all other connected applications to automatically show the relevant data for that client), you can achieve this by using Shared Contexts. A Shared Context is an object holding information in the form of key/value pairs. An application can create, set or update a context object and can also subscribe for changes to that context object in order to react accordingly to context changes and update its information.

## Channels 

You can use the Channels API when you need to offer the user an UI for choosing which apps should be grouped together to synchronize data between each other. Channels are based on Shared Contexts and are UI driven. The user can select from any number of defined color Channels and assign any number and combination of applications to a certain Channel. This way, when the user changes a selection, or updates information in one app, all other apps will synchronize their data according to the user action.

## Pub/Sub  

The Pub/Sub API offers only limited Interop functionality – publishing and subscribing on topics, basic targeting and no streaming. The Pub/Sub support was added to [**Glue42 Enterprise**](https://glue42.com/enterprise/) only for porting apps already using a pub/sub technology. We recommend using all other higher level Interop services offered by [**Glue42 Enterprise**](https://glue42.com/enterprise/) instead of Pub/Sub. 