## Overview

The App Preferences API enables apps to store custom user-specific data and retrieve it when necessary. This allows you to enhance the UX of your apps by instrumenting them to preserve specific user settings and apply them when the app is relaunched.

The App Preferences API provides methods for updating, replacing and clearing user settings stored for the current or a specific app, as well as for all apps of the current user.

## Storage

App preferences can be stored locally in a file, or remotely - using a REST service or the [Glue42 Server](../../glue42-server/index.html). By default, app preferences are stored locally in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\prefs` folder where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) where to store app preferences, use the `"store"` property of the `"applicationPreferences"` top-level key in the `system.json` file located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder:

```json
{
    "applicationPreferences": {
        "store": {
            "type": "rest",
            "restURL": "https://my-rest-service.com",
            "restClientAuth": "no-auth",
            "newDataCheckInterval": 2000
        }
    }
}
```

The `"store"` object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `string` | The type of the app preferences store. Can be one of `"file"` (default), `"rest"` or `"server"`. |
| `"restURL"` | `string` | The URL of the REST service providing the app preferences. Valid only in `"rest"` mode. |
| `"restClientAuth"` | `string` |  The client authentication mechanism for the REST service. Can be one of `"no-auth"` (default), `"negotiate"` or `"kerberos"`. Valid only in `"rest"` mode. |
| `"newDataCheckInterval"` | `number` | Interval in seconds at which to check for new data from the REST store. Executed only if subscribers are available. Valid only in `"rest"` mode. |

The `"type"` property accepts the following values:

| Value | Description |
|-------|-------------|
| `"file"` | Default. App preferences will be saved locally in a file. |
| `"rest"` | App preferences will be saved using the REST service at the URL provided to the `"restURL"` property. |
| `"server"` | App preferences will be saved using a [Glue42 Server](../../glue42-server/index.html) (the [Glue42 Server](../../glue42-server/index.html) must be [configured](../../glue42-server/index.html#how_to-configure_glue42_enterprise) first). |

*For more details, see the [Developers > Configuration](../../../developers/configuration/overview/index.html) section and the [system configuration schema](../../../assets/configuration/system.json).*

For a reference implementation of a remote app preferences store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example. This basic implementation doesn't take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the README file in the repository.