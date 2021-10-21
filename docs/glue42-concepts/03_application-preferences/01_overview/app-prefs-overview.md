## Overview

The Application Preferences API enables applications to store custom user-specific data and retrieve it when necessary. This allows you to enhance the UX of your apps by instrumenting them to preserve specific user settings and apply them when the app is relaunched. 

The Application Preferences API provides methods for updating, replacing and clearing user settings stored for the current or a specific application, as well as for all applications of the current user.

## Storage

Application preferences can be stored locally in a file, or remotely - using a REST service or the [Glue42 Server](../../glue42-server/index.html). By default, application preferences are stored locally in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\prefs` folder where `<ENV-REG>` should be replaced by the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) where to store application preferences, use the `"store"` property of the `"applicationPreferences"` top-level key in the `system.json` file located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder:

```json
{
    "applicationPreferences": {
        "store": {
            "type": "rest",
            "restURL": "https://my-rest-service.com"
        }
    }
}
```

The `"type"` property accepts the following values:

| Value | Description |
|-------|-------------|
| `"file"` | Default. Application preferences will be saved locally in a file. |
| `"rest"` | Application preferences will be saved using the REST service at the URL provided to the `"restURL"` property of the `"store"` key. |
| `"server"` | Application preferences will be saved using the Glue42 Server (the Glue42 Server must be configured first). |

For a reference implementation of a remote application preferences store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example. This basic implementation does not take the user into account and returns the same set of data for all requests. For instructions on running the sample server on your machine, see the `README.md` in the repository.