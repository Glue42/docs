## Referencing

The Glue42 JavaScript library is available as a single JavaScript file, which you can include in your applications. If you are using a Glue42 installer, you can find the JavaScript files in `%LOCALAPPDATA%\Tick42\GlueSDK\GlueJS\js`. When deploying your application in production, it is recommended to always reference a specific minified version:

```javascript
import Glue from "../scripts/desktop.umd.min.js";
```

The Glue42 JavaScript library is also available as an NPM package, which you can include as a dependency in your project and import in your code. The currently available packages are [`@glue42/core`](https://www.npmjs.com/package/@glue42/core) and [`@glue42/desktop`](https://www.npmjs.com/package/@glue42/desktop). The Core package is a subset of the Desktop package and offers basic functionalities for sharing data between applications ([Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html), [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/overview/index.html), [Pub/Sub](../../../../glue42-concepts/data-sharing-between-apps/pub-sub/overview/index.html), [Metrics](../../../../glue42-concepts/metrics/overview/index.html)), while the Desktop package offers additional options for sharing data between apps ([Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html)), as well as advanced window management functionalities ([App Management](../../../../glue42-concepts/application-management/overview/index.html), [Layouts](../../../../glue42-concepts/windows/layouts/overview/index.html), [Window Management](../../../../glue42-concepts/windows/window-management/overview/index.html)).

To include any of the packages as a dependency in your project, navigate to the root directory of your project and run:

```cmd
npm install @glue42/desktop
```

To reference the Glue42 library:

```javascript
import Glue from "@glue42/desktop";
```

## Initialization

If your Node.js app is started by [**Glue42 Enterprise**](https://glue42.com/enterprise/), you can initialize the Glue42 library without passing a configuration object to the factory function:

```javascript
import Glue from "@glue42/desktop";

// No configuration object.
const glue = await Glue().catch(console.error);

console.log(glue.version);
```

<!-- If your Node.js script is started outside [**Glue42 Enterprise**](https://glue42.com/enterprise/), you have to pass a configuration object when initializing Glue42:

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
``` -->

## App Configuration

To add your Node.js app to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with application configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example configuration for a Node.js app:

```json
{
    "name": "node-server",
    "type": "node",
    "service": true,
    "details": {
        "path": "%GDDIR%/PathToMyServer/index.js",
        "showConsole": true
    }
}
```

The `"name"`, `"type"` and `"path"` properties are required and `"type"` must be set to `"node"`. The `"path"` property points to a JavaScript file that [**Glue42 Enterprise**](https://glue42.com/enterprise/) will execute in a Node.js environment.

*For details on how to configure a remotely hosted Node.js app, see the [Developers > Configuration > Application > Node.js > Remote](../../../../developers/configuration/application/index.html#app_configuration-nodejs-remote) section.*

*For details on how to use different Node.js versions, see the [Using Different Node.js Versions](../../../../developers/configuration/application/index.html#app_configuration-nodejs-using_different_nodejs_versions) section.*

## Glue42 JavaScript Concepts

Once the Glue42 JavaScript library has been initialized, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [App Management](../../../../glue42-concepts/application-management/javascript/index.html)
- [App Preferences](../../../../glue42-concepts/application-preferences/javascript/index.html)
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

For a complete list of the available JavaScript APIs, see the [Glue42 JavaScript Reference Documentation](../../../../reference/glue/latest/glue/index.html).