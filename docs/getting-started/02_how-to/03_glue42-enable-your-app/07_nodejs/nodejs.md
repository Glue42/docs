## Initialization 

If your `Node.js` script is started as a Glue42 application, you can initialize the Glue42 library without passing a configuration object to the factory function:

```javascript
const Glue = require("@glue42/desktop");

// No configuration object.
const glue = await Glue().catch(console.error);

console.log(glue.version);
```

If your `Node.js` script is started outside [**Glue42 Enterprise**](https://glue42.com/enterprise/), you have to pass a configuration object when initializing Glue42: 

```javascript
const Glue = require("@glue42/desktop");
const config = {
    application: "MyNodeApp",
    layouts: false,
    appManager: "full",
    logger: false,
    windows: false,
    gateway: {
        protocolVersion: 3,
        ws: process.env.gwURL
    },
    auth: {
        gatewayToken: process.env.gwToken
    }
};

// With configuration object.
const glue = await Glue(config).catch(console.error);

console.log(glue.version);
```

## Application Configuration

To add your app to [**Glue42 Enterprise**](https://glue42.com/enterprise/), you need to create an application configuration JSON file. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` should be replaced by the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example configuration of a `Node.js` app:

```json
{
    "title": "MyNodeServer",
    "name": "nodeServer",
    "caption": "Server description",
    "type": "node",
    "service": true,
    "details": {
        "path": "%GDDIR%/PathToMyServer",
        "showConsole": true
    }
}
```

The only property which is required is the `"path"`. It must lead to a JavaScript file which will be executed by `Node.js` from [**Glue42 Enterprise**](https://glue42.com/enterprise/).

## Glue42 JavaScript Concepts

Once the Glue42 JavaScript library has been initialized, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [Application Management](../../../../glue42-concepts/application-management/javascript/index.html)
- [Application Preferences](../../../../glue42-concepts/application-preferences/javascript/index.html)
- [Intents](../../../../glue42-concepts/intents/javascript/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/javascript/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html)
- [Pub/Sub](../../../../glue42-concepts/data-sharing-between-apps/pub-sub/javascript/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/javascript/index.html)
- [Workspaces](../../../../glue42-concepts/windows/workspaces/javascript/index.html)
- [Layouts](../../../../glue42-concepts/windows/layouts/javascript/index.html)
- [Notifications](../../../../glue42-concepts/notifications/javascript/index.html)
- [Global Search](../../../../glue42-concepts/global-search/index.html)
- [Metrics](../../../../glue42-concepts/metrics/javascript/index.html)
- [Hotkeys](../../../../glue42-concepts/glue42-platform-features/index.html#hotkeys)
- [Zooming](../../../../glue42-concepts/glue42-platform-features/index.html#zooming)
- [Displays](../../../../glue42-concepts/glue42-platform-features/index.html#displays)
- [Logging](../../../../glue42-concepts/glue42-platform-features/index.html#logging)

## Reference

[Glue42 JavaScript Reference](../../../../reference/glue/latest/glue/index.html) 