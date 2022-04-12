## Overview

The Application Management API provides a way to manage [**Glue42 Enterprise**](https://glue42.com/enterprise/) applications. It offers abstractions for:

- *Application* - a program as a logical entity, registered in [**Glue42 Enterprise**](https://glue42.com/enterprise/) with some metadata (name, description, icon, etc.) and with all the configuration needed to spawn one or more instances of it. The Application Management API provides facilities for retrieving application metadata and for detecting when an application is started.

*For details on how to define and configure an application, see the [Application Configuration](../../../developers/configuration/application/index.html) section.*

- *Instance* - a running copy of an application. The Application Management API provides facilities for starting and stopping application instances and tracking application related events.

<glue42 name="diagram" image="../../../images/app-management/app-management.gif">

## Application Stores

[**Glue42 Enterprise**](https://glue42.com/enterprise/) can obtain application configurations from a local store and from a remote REST service. The settings for the application configuration stores are defined in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config`.

In the standard [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment model, application definitions aren't stored locally on the user machine, but are served remotely. If [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured to use a remote application store, it will poll it periodically and discover new application definitions. The store implementation is usually connected to an entitlement system based on which different users can have different applications or versions of the same application. In effect, [**Glue42 Enterprise**](https://glue42.com/enterprise/) lets users run multiple versions of the same application simultaneously and allows for seamless forward/backward application rolling.

[**Glue42 Enterprise**](https://glue42.com/enterprise/) respects the FDC3 standards and can retrieve standard Glue42, as well as FDC3-compliant application definitions. For more details on working with FDC3-compliant apps, see the [FDC3 Compliance](../../../getting-started/fdc3-compliance/index.html) section, the [FDC3 App Directory documentation](https://fdc3.finos.org/docs/app-directory/overview) and the [FDC3 Application](https://fdc3.finos.org/schemas/1.2/app-directory#tag/Application) schema.

<glue42 name="diagram" image="../../../images/app-management/app-stores.png">

### Local App Stores

To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to load application configuration files from a local path, use the `"appStores"` top-level key of the `system.json` file. Set the `"type"` property of the app store configuration object to `"path"` and specify a relative or an absolute path to the application definitions. The environment variables set by [**Glue42 Enterprise**](https://glue42.com/enterprise/) can also be used as values:

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

### Remote App Stores

Application configurations can also be hosted on a server and obtained from a REST service.

For a reference implementation of a remote application configurations store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example that implements the [FDC3 App Directory](https://fdc3.finos.org/docs/1.0/appd-intro) and is compatible with [**Glue42 Enterprise**](https://glue42.com/enterprise/). This basic implementation doesn't take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the README file in the repository.

For a .NET implementation of a remote application configurations store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

To configure a connection to the REST service providing the application store, add a new entry to the `"appStores"` top-level key of the `system.json` file:

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

The only required properties for each app store configuration object are `"type"`, which should be set to `"rest"`, and `"url"`, which is the address of the remote application store. You can also set the authentication, polling interval, cache persistence and cache folder.

| Property | Description |
|----------|-------------|
| `"auth"` | Authentication configuration. Can be one of `"no-auth"`, `"negotiate"` or `"kerberos"`. |
| `"pollInterval"` | Interval in milliseconds at which to poll the REST service for updates. |
| `"enablePersistentCache"` | Whether to cache and persist the configuration files locally (e.g., in case of connection interruptions). |
| `"cacheFolder"` | Where to keep the persisted configuration files. |

The remote store must return application definitions in the following response shape:

```json
{
    "applications": [
        // List of application definition objects.
        {}, {}
    ]
}
```

You can also use the [Glue42 Server](../../glue42-server/index.html) for hosting and retrieving application stores. The [Glue42 Server](../../glue42-server/index.html) is a complete server-side solution for providing data to Glue42. To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to fetch application configurations from a [Glue42 Server](../../glue42-server/index.html), set the `"type"` property of the app store configuration object to `"server"`:

```json
{
    "appStores": [
        {
            "type": "server"
        }
    ]
}
```