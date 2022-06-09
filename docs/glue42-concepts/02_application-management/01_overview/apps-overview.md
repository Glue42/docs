## Overview

The App Management API provides a way to manage [**Glue42 Enterprise**](https://glue42.com/enterprise/) apps. It offers abstractions for:

- *App* - a program as a logical entity, registered in [**Glue42 Enterprise**](https://glue42.com/enterprise/) with some metadata (name, description, icon, etc.) and with all the configuration needed to spawn one or more instances of it. The App Management API provides facilities for retrieving app metadata and for detecting when an app is started.

*For details on how to define and configure an app, see the [App Configuration](../../../developers/configuration/application/index.html) section.*

- *Instance* - a running copy of an app. The App Management API provides facilities for starting and stopping app instances and tracking app-related events.

<glue42 name="diagram" image="../../../images/app-management/app-management.gif">

## App Stores

[**Glue42 Enterprise**](https://glue42.com/enterprise/) can obtain app configurations from a local store, from a remote REST service, or from a [Glue42 Server](../../glue42-server/index.html). The settings for the app configuration stores are defined in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config`, under the `"appStores"` top-level key, which accepts an array of objects defining one or more app stores.

In the standard [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment model, app definitions aren't stored locally on the user machine, but are served remotely. If [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured to use a remote app store, it will poll it periodically and discover new app definitions. The store implementation is usually connected to an entitlement system based on which different users can have different apps or versions of the same app. In effect, [**Glue42 Enterprise**](https://glue42.com/enterprise/) lets users run multiple versions of the same app simultaneously and allows for seamless forward/backward app rolling.

[**Glue42 Enterprise**](https://glue42.com/enterprise/) respects the FDC3 standards and can retrieve standard Glue42, as well as FDC3-compliant app definitions. For more details on working with FDC3-compliant apps, see the [FDC3 Compliance](../../../getting-started/fdc3-compliance/index.html) section, the [FDC3 App Directory documentation](https://fdc3.finos.org/docs/app-directory/overview) and the [FDC3 Application](https://fdc3.finos.org/schemas/1.2/app-directory#tag/Application) schema.

<glue42 name="diagram" image="../../../images/app-management/app-stores.png">

### Local

To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to load app configuration files from a local path, set the `"type"` property of the app store configuration object to `"path"` and specify a relative or an absolute path to the app definitions. The environment variables set by [**Glue42 Enterprise**](https://glue42.com/enterprise/) can also be used as values:

```json
{
    "appStores": [
        {
            "type": "path",
            "details": {
                "path": "../config/apps"
            }
        },
        {
            "type": "path",
            "details": {
                "path": "%GD3-APP-STORE%"
            }
        },
        {
            "type": "path",
            "details": {
                "path": "%GLUE-USER-DATA%/apps"
            }
        }
    ]
}
```

Each app store object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `string` | **Required.** Type of the app store. Must be set to `"path"` for local path app stores. |
| `"details"` | `object` | **Required.** Specific details about the app store. |

The `"details"` object for a local path app store has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"path"` | `string` | **Required.** Must point to the local app store. The specified path can be absolute or relative and you can use defined environment variables. |

### Remote

App configurations can also be obtained from remote app stores via a REST service.

For a reference implementation of a remote app configurations store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example that implements the [FDC3 App Directory](https://fdc3.finos.org/docs/appd-intro) and is compatible with [**Glue42 Enterprise**](https://glue42.com/enterprise/). This basic implementation doesn't take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the README file in the repository.

For a .NET implementation of a remote app configurations store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

To configure a connection to the REST service providing the remote app store, add a new entry to the `"appStores"` top-level key of the `system.json` file and set its `"type"` to `"rest"`:

```json
{
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
}
```

Each app store object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `string` | **Required.** Type of the app store. Must be set to `"rest"` for remote app stores. |
| `"details"` | `object` | **Required.** Specific details about the app store. |
| `"isRequired"` | `boolean` | If `true` (default), the app store will be required. If the app store can't be retrieved, [**Glue42 Enterprise**](https://glue42.com/enterprise/) will throw an error and shut down. If `false`, [**Glue42 Enterprise**](https://glue42.com/enterprise/) will initiate normally, without apps from that store. |

The `"details"` object for a remote app store has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"url"` | `string` | **Required.** The URL to the REST service providing the app configurations. |
| `"auth"` | `string` | Authentication configuration. Can be one of `"no-auth"` (default), `"negotiate"` or `"kerberos"`. |
| `"pollInterval"` | `number` | Interval in milliseconds at which to poll the REST service for updates. Default is `60000`. |
| `"enablePersistentCache"` | `boolean` | If `true` (default), will cache and persist the configuration files locally (e.g., in case of connection interruptions). |
| `"cacheFolder"` | `string` | Where to keep the persisted configuration files. |
| `"readCacheAfter"` | `number` | Interval in milliseconds after which to try to read the cache. Default is `30000`. |
| `"startRetries"` | `number` | Number of times [**Glue42 Enterprise**](https://glue42.com/enterprise/) will try to connect to the REST server. Default is `5`. |
| `"startRetryInterval"` | `number` | Interval in milliseconds at which [**Glue42 Enterprise**](https://glue42.com/enterprise/) will try to connect to the REST Server. Default is `10000`. |
| `"requestTimeout"` | `number` | Timeout in milliseconds to wait for a response from the REST server. Default is `20000`. |
| `"proxy"` | `string` | HTTP proxy to use when fetching data. |
| `"rejectUnauthorized"` | `boolean` | If `true` (default), SSL validation will be enabled for the REST server. |

The remote store must return app definitions in the following response shape:

```json
{
    "applications": [
        // List of app definition objects.
        {}, {}
    ]
}
```

### Glue42 Server

You can also use the [Glue42 Server](../../glue42-server/index.html) for hosting and retrieving app stores. The [Glue42 Server](../../glue42-server/index.html) is a complete server-side solution for providing data to Glue42. To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to fetch app configurations from a [Glue42 Server](../../glue42-server/index.html), set the `"type"` property of the app store configuration object to `"server"`:

```json
{
    "appStores": [
        {
            "type": "server"
        }
    ]
}
```

Each app store object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `string` | **Required.** Type of the app store. Must be set to `"server"` for app stores retrieved from a [Glue42 Server](../../glue42-server/index.html). |

If you are using only a [Glue42 Server](../../glue42-server/index.html) for retrieving app configurations, you can set the `"appStores"` key to an empty array. [**Glue42 Enterprise**](https://glue42.com/enterprise/) will automatically try to connect to the [Glue42 Server](../../glue42-server/index.html) using the [configuration](../../glue42-server/index.html#how_to-configure_glue42_enterprise) for it, and will retrieve the app configurations from it, if any.