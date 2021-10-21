## Overview

[FDC3](https://fdc3.finos.org/) aims at developing specific protocols and classifications in order to advance the ability of desktop applications in financial workflows to interoperate in a plug-and-play fashion without prior bilateral agreements.

## FDC3 for Glue42 Enterprise

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

Below are explained the specifics for running an FDC3 compliant app within [**Glue42 Enterprise**](https://glue42.com/enterprise/). For a detailed information on the FDC3 API itself, see the [FDC3 documentation](https://fdc3.finos.org/docs/next/api/overview).

### Configuration

To deploy your FDC3 compliant application in [**Glue42 Enterprise**](https://glue42.com/enterprise/), you have to apply a few configuration settings in order to include your app in [**Glue42 Enterprise**](https://glue42.com/enterprise/) and start using the FDC3 API. It is not necessary to reference an FDC3 library, as [**Glue42 Enterprise**](https://glue42.com/enterprise/) provides configurable injection of an FDC3 API implementation - the [`@glue42/fdc3`](https://www.npmjs.com/package/@glue42/fdc3) library.

An important note about the auto injected Glue42 FDC3 library is that it uses internally the [Glue42 JavaScript library](../how-to/glue42-enable-your-app/javascript/index.html) and initializes only the Glue42 APIs on which it depends - [Interop](../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html), [Shared Contexts](../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html), [Channels](../../glue42-concepts/data-sharing-between-apps/channels/javascript/index.html), [Application Management](../../glue42-concepts/application-management/javascript/index.html) and [Intents](../../glue42-concepts/intents/javascript/index.html). You must always consider this if your app also initializes Glue42, because the Glue42 JavaScript library is optimized to reuse an already existing Glue42 instance (if any) instead of initializing again. The Glue42 FDC3 library is initialized before the execution of your application code, which means that if the Glue42 FDC3 library is configured to initialize Glue42, your app will receive that Glue42 instance. This is significant in situations where your app depends on other Glue42 APIs as well - [Layouts](../../glue42-concepts/windows/layouts/javascript/index.html), [Window Management](../../glue42-concepts/windows/window-management/javascript/index.html), etc. Use the `"fdc3InitsGlue"` property of the `"details"` top-level key in your application configuration to control whether the Glue42 FDC3 library will return a Glue42 instance. By default, it is set to `false` which means that your app will be responsible for initializing Glue42. Keep in mind that the Glue42 instance provided by your app must include all Glue42 APIs on which the Glue42 FDC3 library depends, otherwise it will throw an error. In case you want to use only the Glue42 FDC3 API implementation, or you are satisfied with the Glue42 APIs that the Glue42 FDC3 library offers, set the `"fdc3InitsGlue"` property to `true`.

To enable injecting the Glue42 FDC3 API implementation, edit your application configuration and the system configuration of [**Glue42 Enterprise**](https://glue42.com/enterprise/):

- Use the `"autoInjectFdc3"` and `"fdc3InitsGlue"` properties of the `"details"` top-level key in your application configuration to enable auto injecting the Glue42 FDC3 library and to configure it whether to initialize Glue42 or not:

```json
{
    "title": "My FDC3 App",
    "type": "window",
    "name": "my-fdc3-app",
    "hidden": false,
    "details": {
        "url": "http://localhost:8047/index.html",
        "autoInjectFdc3": {
            "enabled": true
        },
        "fdc3InitsGlue": true
    }
}
```

*For more information on configuring your apps for [**Glue42 Enterprise**](https://glue42.com/enterprise/), see the [Application Configuration](../../developers/configuration/application/index.html) section.*

*FDC3 configuration standards are supported by [**Glue42 Enterprise**](https://glue42.com/enterprise/), so you can also supply the above Glue42 configuration JSON file as a string to the `manifest` property of an [FDC3 application configuration](https://fdc3.finos.org/schemas/1.1/app-directory#tag/Application) file.*

- Next, configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to auto inject the FDC3 API and specify which apps to inject with it. Open the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) (located in `%LocalAppData%\Tick42\GlueDesktop\config`) and set the `"autoInjectFdc3"` property under the `"windows"` top-level key:

```json
{
    "windows": {
        "autoInjectFdc3": {
            "enabled": true,
            "version": "*",
            "whitelist": ["my-fdc3-app"]
        }
    }
}
```

Now the FDC3 API will be available globally as a property of the `window` object.

```javascript
const handler = context => console.log(`Context: ${JSON.stringify(context)}`);

fdc3.addContextListener(handler);
```

### Intents

The [FDC3 Intents](https://fdc3.finos.org/docs/next/intents/overview) concept serves the purpose of enabling the creation of cross-application workflows on the desktop. An application declares an Intent through configuration. An Intent specifies what action the application can execute and what data structure it can work with.

Intents can be defined both in the `"intents"` top-level array of an [FDC3 application configuration](https://fdc3.finos.org/schemas/1.1/app-directory#tag/Application) file, or in a Glue42 application configuration file.

Application configuration files can be stored either remotely (e.g., in an FDC3-compliant [App Directory](#fdc3_for_glue42_enterprise-app_directory)) or locally. Local user defined application files are usually located in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` should be replaced by the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). Intents can be configured under the `"intents"` top-level key of the application configuration file.

Below is an example configuration for an Intent:

```json
"intents": [
    {
        "name": "ShowChart",
        "displayName": "BBG Instrument Chart",
        "contexts": ["Instrument"]
    }
]
```

- `"name"` - Required. The name of the Intent;
- `"displayName"` - The human readable name of the Intent. Can be used in context menus, etc., to visualize the Intent;
- `"contexts"` - Required. The type of predefined data structures that the application can work with (see [FDC3 Contexts](https://fdc3.finos.org/docs/next/context/overview)).

*For more information on using Intents, see the [FDC3 Intents API](https://fdc3.finos.org/docs/next/intents/overview).*

### Channels

An [FDC3 Channel](https://fdc3.finos.org/docs/next/api/ref/Channel) is a named context object that an application can join in order to share and update context data and also be alerted when the context data changes. By [specification](https://fdc3.finos.org/docs/next/api/spec#context-channels), Channels can either be well-known system Channels (one of which is called "default") or Channels created by apps. On a UI level, Channels can be represented by colors and names.

All system defined Channels in [**Glue42 Enterprise**](https://glue42.com/enterprise/) can be found in the `channels.json` file in the `%LocalAppData%\Tick42\GlueDesktop\config` folder. There you can easily define as many custom Channels as you want. For instance, to add a purple Channel to the existing list of system Channels, you need to add the following configuration:

```json
{
    "name": "Purple",
    "meta": {
        "color": "#6400b0"
    }
}
```

To add a Channel Selector (UI component from which the user can manually switch between Channels) to your window, you need to set `"allowChannels"` to `true` in your application configuration file, under the `"details"` top-level key:

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

*For more information on using Channels, see the [FDC3 Channels API](https://fdc3.finos.org/docs/next/api/ref/Channel).*

### App Directory

The goal of the [FDC3 App Directory](https://fdc3.finos.org/docs/next/app-directory/overview) REST service is to provide trusted identity for desktop apps. Application definitions are provided by one or more App Directory REST services where user entitlements and security can also be handled.

To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to retrieve application definitions from remote application stores, you only need to add a new entry to the `"appStores"` top-level key of the `system.json` file located in the `%LOCALAPPDATA%\Tick42\GlueDesktop\config` folder:

```json
"appStores": [
    {
        "type": "rest",
        "details": {
            "url": "http://localhost:3000/appd/v1/apps/search",
            "auth": "no-auth",
            "pollInterval": 30000,
            "enablePersistentCache": true,
            "cacheFolder": "%LocalAppData%/Tick42/UserData/%GLUE-ENV%-%GLUE-REGION%/gcsCache/"
        }
    }
]
```

- `"auth"` - authentication configuration;
- `"pollInterval"` - interval at which to poll the REST service for updates;
- `"enablePersistentCache"` - whether to cache and persist the layouts locally (e.g., in case of connection interruptions);
- `"cacheFolder"` - where to keep the persisted layout files;

Note that currently only [application configurations](../../developers/configuration/application/index.html) that comply with Glue42 specifications are supported.

*For a reference implementation of the FDC3 App Directory, see the [Node.js REST Config Example](https://github.com/Glue42/rest-config-example-node-js) on GitHub.*

*For more information on using App Directory, see the [FDC3 App Directory documentation](https://fdc3.finos.org/docs/next/app-directory/overview).* 