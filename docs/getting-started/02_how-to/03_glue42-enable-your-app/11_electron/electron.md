## Overview

All Glue42 [JavaScript](../javascript/index.html) functionalities are available for your Electron apps through the Glue42 Electron library - [`@glue42/electron`](https://www.npmjs.com/package/@glue42/electron).

Your Glue42 enabled Electron apps can be configured as Glue42 applications (see [Application Configuration](#application_configuration)) in order to be started by [**Glue42 Enterprise**](https://glue42.com/enterprise/), or they can run independently.

When initialized in your Electron app, the [`@glue42/electron`](https://www.npmjs.com/package/@glue42/electron) library will discover any running instance of Glue42 and connect to it and will inject the Glue42 library in all windows of your app. Registering your main window as a [Glue42 Window](../../../../glue42-concepts/windows/window-management/overview/index.html) will enable it to stick to other Glue42 Windows, use [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) and be saved and restored in [Layouts](../../../../glue42-concepts/windows/layouts/overview/index.html). The library allows your app to register itself as an [app factory](#registering_app_windows-app_factories) for certain types of windows and also to register its own [child windows](#registering_app_windows-child_windows) as Glue42 apps so that they can participate in Glue42.

## Referencing and Initialization

The [`@glue42/electron`](https://www.npmjs.com/package/@glue42/electron) library is available as an NPM package. To install it, run the following command in the root directory of your project:

```cmd
npm install @glue42/electron
```

Reference the library in your Electron app and use the `initialize()` method to initialize the library in the main process of your app, after the app `"ready"` event. The `glue` object returned by a successful initialization is the entry point for all Glue42 APIs:

```javascript
// Reference the Glue42 Electron library.
import * as glue42Electron from "@glue42/electron";

// Initialize the library to access the Glue42 APIs.
const glue = await glue42Electron.initialize();
```

### Configuration

The Glue42 Electron library can also be initialized with an optional configuration object:

```javascript
// Provide optional configuration for your app.
const config = {
    appDefinition: {
        name: "my-electron-app",
        title: "My Electron App"
    }
};

const glue = await glue42Electron.initialize(config);
```

The configuration object has the following properties, all of which are optional:

| Property | Type | Description |
|----------|------|-------------|
| `appDefinition` | `object` | Runtime definition for an EXE app. For more details, see [Application Configuration](#application_configuration). |
| `env` | `string` | [**Glue42 Enterprise**](https://glue42.com/enterprise/) environment. |
| `region` | `string` | [**Glue42 Enterprise**](https://glue42.com/enterprise/) region. |
| `gwURL` | `string` | URL to the Glue42 Gateway to which to connect. |
| `inject` | `"glue"` \| `"fdc3"` \| `"none"` | By default, the Glue42 Electron library will inject the [`@glue42/desktop`](https://www.npmjs.com/package/@glue42/desktop) library in your Electron app windows. To inject the [`@glue42/fdc3`](https://www.npmjs.com/package/@glue42/fdc3) library instead, set to `"fdc3"`. To disable injection, set to `"none"`. Note that the libraries will be injected, but not initialized. If you want to inject a library, the `preload` property must be set to `true`. |
| `preload` | `boolean` | When `true` (default), a service object will be added to the list of preloads for each window. This object holds information which is necessary for connecting to Glue42. Set to `false` if you want to disable adding preloads to your windows - in this case, injection will be disabled and you will have to reference the Glue42 library in your windows in order to be able to use it. You will also have to provide the necessary information for connecting to Glue42. This information can be extracted from the [Startup Options](#referencing_and_initialization-startup_options) when the app is started by [**Glue42 Enterprise**](https://glue42.com/enterprise/), or in the case of [registering an app factory](#registering_app_windows-app_factories), you can use the service object that is passed as an argument to the factory function. |

### Startup Options

When your Electron app is started by [**Glue42 Enterprise**](https://glue42.com/enterprise/), you can extract the application startup options from the `glue` object returned by the initialized Glue42 Electron library:

```javascript
// Extracting the startup options.
const options = glue.startupOptions;

// Extracting the application starting context.
const context = options.context;

// Extracting the Glue42 Window settings.
const windowSettings = options.windowOptions;
```

The startup options object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `instanceId` | `string` | Instance ID. |
| `context` | `object` | Startup context for the new application. |
| `gwURL` | `string` | URL to the Glue42 Gateway to which to connect. |
| `gwToken` | `string` | Token to be used as an authentication mechanism when connecting to the Glue42 Gateway. |
| `applicationConfig` | `object` | The app configuration as defined in the application store. See [Application Configuration](#application_configuration). |
| `env` | `string` | The [**Glue42 Enterprise**](https://glue42.com/enterprise/) environment. |
| `region` | `string` | The [**Glue42 Enterprise**](https://glue42.com/enterprise/) region. |
| `windowOptions` | `object` | [Glue42 Window](../../../../glue42-concepts/windows/window-management/overview/index.html) settings. |

## Registering App Windows

The Glue42 Electron library allows you to register your Electron app windows in the Glue42 framework dynamically at runtime. This will enable them to stick to other [Glue42 Windows](../../../../glue42-concepts/windows/window-management/overview/index.html), use [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) and be saved and restored in [Layouts](../../../../glue42-concepts/windows/layouts/overview/index.html).

*For details on how to configure your Electron app as a Glue42 app using configuration files, see the [Application Configuration](#application_configuration) section.*

### Main Window

When your app is ready and the Glue42 Electron library has been initialized, register your main window using the `registerStartupWindow()` method. Pass the main window as a required first argument and, optionally, pass a configuration object that will override the [application configuration](#application_configuration) (if any).

The following example demonstrates how to register the main window, enable the Glue42 [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html) and join the `"Red"` Channel:

```javascript
// Optional configuration for your main window.
const config = {
    allowChannels: true,
    channelId: "Red"
};

// Register your main window.
const glue42MainWindow = await glue.registerStartupWindow(this.mainWindow, config);
```

### Child Windows

Your Electron app may be able to create addition windows offering different functionalities to the user - e.g., a chat window that can be opened by clicking a button or a link in your main window. You can register this child window as a Glue42 app so that it will be able to participate fully in Glue42.

To register a child window, use the `registerChildWindow()` method. Pass a browser window and an app definition object as the first two required arguments and, optionally, specify Glue42 Window settings.

The following example demonstrates how to register a child window as a tab window and specify its bounds:

```javascript
const bw = new BrowserWindow();

// Provide an application definition.
const appDefinition = {
    name: "my-child-electron-app",
    title: "My Child Electron App"
};

// Provide window options.
const options = {
    mode: "tab",
    left: 100,
    top: 100,
    width: 400,
    height: 400
};

// Register your child window.
const glue42Window = await glue.registerChildWindow(bw, appDefinition, options);
```

### App Factories

You can register your Electron app as a factory for different types of windows. This will allow any other Glue42 app to create that window with a given context - e.g., open a chat window and pass relevant information to it.

To register an app factory, use the `registerAppFactory()` method which accepts an application definition and a factory function as required parameters. The factory function will be invoked with three arguments - the provided application definition, a context object (startup context or last saved context when restoring the window) and a service object that holds information about the current Glue42 environment which is necessary for connecting to Glue42:

```javascript
// Provide an application definition.
const appDefinition = {
    name: "my-child-electron-app",
    title: "My Child Electron App"
};

// Provide a factory function for creating the app.
const factory = (appDefinition, context, glue42electron) => {

    // If you want to override window settings, use `this`.
    this.title = context.title;

    const bw = new BrowserWindow();

    // Must return the newly created browser window.
    return bw;
};

// Register the application factory.
await glue.registerAppFactory(appDefinition, factory);
```

*Note that you can use the service object passed as a third argument to the factory function to properly initialize the Glue42 Electron library in your window in case you have disabled preloads. For more details, see the [Configuration](#referencing_and_initialization-configuration) section.*

## Application Configuration

To add your Electron app to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with application configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

The following is an example configuration for an Electron app:

```json
{
    "title": "My Electron App",
    "type": "exe",
    "name": "my-electron-app",
    "icon": "https://example.com/icon.ico",
    "details": {
        "path": "%GDDIR%/../Demos/MyElectronApp/",
        "command": "MyElectronApp.exe",
        "parameters": " --mode=1"
    }
}
```

| Property | Description |
|----------|-------------|
| `"type"` | Must be `"exe"`. |
| `"path"` | The path to the application - relative or absolute. You can also use the `%GDDIR%` environment variable, which points to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installation folder. |
| `"command"` | The actual command to execute (the EXE file name). |
| `"parameters"` | Specifies command line arguments. |

For more detailed information about the application definitions, see the [Configuration](../../../../developers/configuration/application/index.html#application_configuration-exe) documentation.

*See the [Electron example](https://github.com/Glue42/electron-example) on GitHub which demonstrates the various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

## Glue42 JavaScript Concepts

Once the Glue42 Electron library has been initialized, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

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

For a complete list of the available JavaScript APIs, see the [Glue42 JavaScript Reference Documentation](../../../../reference/glue/latest/glue/index.html).