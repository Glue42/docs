## Overview

The Application Preferences API enables applications to store custom user-specific data and retrieve it when necessary. This allows you to enhance the UX of your apps by instrumenting them to preserve specific user settings and apply them when the app is relaunched.

The Application Preferences API provides methods for updating, replacing and clearing user settings stored for the current or a specific application, as well as for all applications of the current user.

## Storage

Application preferences can be stored locally in a file, or remotely - using a REST service or the [Glue42 Server](../../glue42-server/index.html). By default, application preferences are stored locally in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\prefs` folder where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) where to store application preferences, use the `"store"` property of the `"applicationPreferences"` top-level key in the `system.json` file located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder:

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

The `"store"` key has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `"file"` \| `"rest"` \| `"server"` | The type of the app preferences store. |
| `"restURL"` | `string` | The URL of the REST service providing the app preferences. Valid only in `"rest"` mode. |
| `"restClientAuth"` | `"no-auth"` \| `"negotiate"` \| `"kerberos"` |  The client authentication mechanism for the REST service. Valid only in `"rest"` mode. |
| `"newDataCheckInterval"` | `number` | Interval in seconds at which to check for new data from the REST or server store. Executed only if subscribers are available. |

The `"type"` property accepts the following values:

| Value | Description |
|-------|-------------|
| `"file"` | Default. Application preferences will be saved locally in a file. |
| `"rest"` | Application preferences will be saved using the REST service at the URL provided to the `"restURL"` property. |
| `"server"` | Application preferences will be saved using the Glue42 Server (the Glue42 Server must be configured first). |

*For more details, see the [Developers > Configuration](../../../developers/configuration/overview/index.html) section and the [system configuration schema](../../../assets/configuration/system.jsons).*

For a reference implementation of a remote application preferences store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example. This basic implementation doesn't take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the README file in the repository.