## Application Configuration

The configuration of an application is a JSON file which allows the application to be accessible to the user from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html). It consists of base properties which are common for all types of applications, and type-specific properties located under the `"details"` top-level key. The required properties for all types of applications are `"name"`, `"type"` and `"details"`.

The custom user application configurations should be stored in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). The files at this location won't be erased or overwritten in case you upgrade your version of [**Glue42 Enterprise**](https://glue42.com/enterprise/).

The configuration files of the apps that come with the [**Glue42 Enterprise**](https://glue42.com/enterprise/) installer are located in the `%LocalAppData%\Tick42\GlueDesktop\config\apps` folder. For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### Window

Window applications are web applications which run in a Glue42 browser window.

```json
{
    "name":"glue42-website",
    "title":"Glue42 Website",
    "icon": "https://example.com/icon.ico",
    "type": "window",
    "details":{
        "url":"https://glue42.com",
        "mode": "tab",
        "width": 400,
        "height": 400
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"window"`. The `"url"` property points to the location of the web application.

The value of the `"title"` property will be used as a window title and also as a name for the application in the Glue42 Toolbar. The `"mode"` property defines the Glue42 Window mode - `"flat"` (default), `"tab"` or `"html"`.

*For the differences between the Glue42 Window modes, see the [Window Modes](../../../glue42-concepts/windows/window-management/overview/index.html#window_modes) section.*

The supported formats for the application icon are ICO, PNG, APNG and JPG.

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### Exe

An executable application which can be executed from your OS. This is a basic example for an EXE configuration:

```json
{
    "name":"My EXE App",
    "title":"My App",
    "type": "exe",
    "details":{
         "path":"%GDDIR%/../PathToWPFApplication/",
         "command": "WPFApplication.exe",
         "parameters": "param1 param2",
         "width": 400,
         "height": 400
    }
}
```

*Note that currently size constraints properties (e.g., `"width"`, `"height"`, `"maxWidth"`, `"minHeight"`, etc.) are only valid for EXE applications that are registered as Glue42 Windows.*

The `"name"`, `"type"` and `"path"` properties are required and `"type"` must be set to `"exe"`. The `"path"` property points to the application working directory. The `"command"` property accepts the actual command to execute - the EXE file name. To specify command line parameters for starting the application, use the `"parameters"` property.

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### ClickOnce

The following example demonstrates how to configure a Glue42 enabled ClickOnce application:

```json
{
    "title": "My ClickOnce App",
    "type": "clickonce",
    "name": "my-clickonce-app",
    "details": {
        "url": "https://example.com/my-clickonce-app.application",
        "width": 1000,
        "height": 400,
        "appParameters": [
            {
                "name": "p1",
                "value": "customParameter"
            }
        ]
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"clickonce"`. The `"url"` property points to the physical location where the ClickOnce application is deployed and from where it will be installed on the user machine.

The `"appParameters"` property is an array of objects defining custom parameters that your application can access at runtime through `glue.GDStartingContext.ApplicationConfig.Details.AppParameters`. Each object sets the `"name"` and the `"value"` of a custom parameter.

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### Batch file

Batch files can also be included as [**Glue42 Enterprise**](https://glue42.com/enterprise/) applications.

The following example demonstrates how to configure a batch file:

```json
{
    "title":"My Batch File",
    "name":"my-batch-file",
    "type": "exe",
    "details":{
         "path":"%GDDIR%/../PathToMyBatchFile/",
         "command": "my-batch-file.bat",
         "parameters": "param1 param2",
         "width": 400,
         "height": 400
    }
}
```

The `"name"`, `"type"` and `"path"` properties are required and `"type"` must be set to `"exe"`. The `"path"` property points to the application working directory. The `"command"` property accepts the actual command to execute - the BAT file name. To specify command line parameters for starting the application, use the `"parameters"` property.

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### Node.js

Both locally and remotely hosted applications running in a Node.js environment can participate in Glue42. It is also possible to specify different Node.js versions for the different apps.

#### Local

The following example demonstrates how to configure an application which runs locally in a Node.js environment:

```json
{
    "name": "node-server",
    "type": "node",
    "service": true,
    "details": {
        "path": "%GDDIR%/PathToMyServer/index.js",
        "nodeVersion": "16.13.2",
        "showConsole": true
    }
}
```

The `"name"`, `"type"` and `"path"` properties are required and `"type"` must be set to `"node"`. The `"path"` property points to a JavaScript file that [**Glue42 Enterprise**](https://glue42.com/enterprise/) will execute in a Node.js environment.

Use the `"nodeVersion"` property to specify a Node.js version for the application.

*For more details on how to name and where to place the different Node.js versions, see the [Using Different Node.js Versions](#application_configuration-nodejs-using_different_nodejs_versions) section).*

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

#### Remote

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.14">

*Note that remotely hosted Node.js apps are subject to certain restrictions when participating in Glue42. [**Glue42 Enterprise**](https://glue42.com/enterprise/) will fetch and execute only the file specified in the app configuration, so your remote app must be bundled into a single file and must not depend on any packages with native code that can't be bundled with it.*

The following example demonstrates how to configure an application which runs remotely in a Node.js environment:

```json
{
    "name": "node-server",
    "type": "node",
    "service": true,
    "details": {
        "remote": {
            "url": "https://example.com/node-server/bundled.js",
            "noCache": true
        },
        "showConsole": true
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"node"`. The `"url"` property points to a single bundled JavaScript file that [**Glue42 Enterprise**](https://glue42.com/enterprise/) will fetch and execute in a Node.js environment.

The `"service"` property can be used to specify that this is a service app and shouldn't be closed when saving and restoring a [Layout](../../../glue42-concepts/windows/layouts/overview/index.html). The `"showConsole"` property can be used to specify whether the Node.js console should be visible or not.

The `"remote"` object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"url"` | `string` | The URL to the JavaScript file to be fetched and executed in Node.js. |
| `"headers"` | `object[]` | Set of headers that will be added to the request for downloading the script file. Each object in the array must have valid `"name"` and `"value"` properties holding the header name and the respective value. |
| `"noCache"` | `boolean` | If `true`, the `Cache-control: no-cache` header will be added. |

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

#### Using Different Node.js Versions

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.14">

[**Glue42 Enterprise**](https://glue42.com/enterprise/) comes with a default Node.js version, but also allows you to specify a Node.js version per application using the `"nodeVersion"` property of the application configuration. The default Node.js environment is named `node.exe` and is located in the `%LocalAppData%\Tick42\GlueDesktop\assets\node` folder. The default path to the Node.js environment is specified using the `"nodePath"` top-level key in the [`system.json`](../../../assets/configuration/system.json) configuration file of [**Glue42 Enterprise**](https://glue42.com/enterprise/). All additional Node.js executable files must be placed in the same specified folder and their names must be in the format `node-[version].exe` (e.g., `node-16.13.2.exe`), in order for [**Glue42 Enterprise**](https://glue42.com/enterprise/) to be able to find the correct version for each Node.js app.

### Citrix App

A Citrix Virtual Application can be configured just like any other Glue42 enabled application. It can participate in the Glue42 environment as a first-class citizen and use all Glue42 functionalities. A Citrix application must be configured as a `"citrix"` type and the `"name"` property in the `"details"` key must be set with the exact published name of the Citrix application. Use the `"parameters"` property under `"details"` to pass command line arguments for starting the app.

The following demonstrates a basic Citrix app configuration:

```json
{
    "title": "Client List - Citrix",
    "type": "citrix",
    "name": "clientlist-citrix",
    "details": {
        "name": "Client List",
        "parameters": "--mode=3 --clients=http://localhost:22060/clients",
        "left": 100,
        "top": 100,
        "width": 600,
        "height": 700
    }
}
```

*For system-wide Citrix configuration, see the [System Configuration](../system/index.html#citrix_apps) section.*

### Service Window

Service windows are a specific usage of a window application. They can provide data and enhance other applications throughout the [**Glue42 Enterprise**](https://glue42.com/enterprise/) life cycle. The service window is defined as an application that is hidden and may be auto started when [**Glue42 Enterprise**](https://glue42.com/enterprise/) is initiated.

There are different ways to configure a service window, depending on whether you want the window to be automatically started when [**Glue42 Enterprise**](https://glue42.com/enterprise/) is initiated. Use a combination of the following application configuration properties to specify whether the window should be automatically started, invisible, or hidden from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html):

| Property | Type | Description |
|----------|------|-------------|
| `"service"` | `boolean` | If `true`, both the `"autoStart"` top-level key and the `"hidden"` property of the `"details"` object will be overridden and set to `true`. The window will be invisible and will start automatically on [**Glue42 Enterprise**](https://glue42.com/enterprise/) startup. |
| `"hidden"` | `boolean` | If `true`, the application won't be available in the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html). |
| `"autoStart"` | `boolean` | If `true`, the window will be started automatically on [**Glue42 Enterprise**](https://glue42.com/enterprise/) startup. |
| `"details"` | `object` | Use the `"hidden"` Boolean property of the `"details"` object to set the window visibility. If `true`, the window will be invisible. |

*Note that when using the `"service"` property, it's pointless to use the `"autoStart"` top-level key and the `"hidden"` property of the `"details"` object, because the `"service"` key will override any values you may set for them.*

The following example demonstrates how to use the `"service"` and `"hidden"` top-level keys to configure a service window that will start automatically, will be invisible and hidden from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html):

```json
{
    "name": "service-window",
    "type": "window",
    "service": true,
    "hidden": true,
    "details": {
        "url": "https://example.com/my-service-window",
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"window"`. The `"url"` property points to the location of the web application.

The following example demonstrates how to use the `"hidden"` and `"autoStart"` top-level keys and the `"hidden"` property of the `"details"` top-level key to configure a service window that will be hidden from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html), won't start automatically and will be invisible:

```json
{
    "name": "service-window",
    "type": "window",
    "hidden": true,
    "autoStart": false,
    "details": {
        "url": "https://example.com/my-service-window",
        "hidden": true
    }
}
```

The `"hidden"` property in the `"details"` object will make the window invisible, while the `"hidden"` top-level key will hide the application from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html) so that it won't be accessible to the user.

*Note that service windows aren't closed when restoring a [Layout](../../../glue42-concepts/windows/layouts/overview/index.html).*

For more details, see the [application configuration schema](../../../assets/configuration/application.json).

### Frameless Window

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.15">

Frameless windows are based on HTML windows, but allow for creating applications with freeform (non-rectangular) shapes and transparent areas. They don't have the usual [Glue42 Window](../../../glue42-concepts/windows/window-management/overview/index.html) decorations - title bars, standard system buttons ("Minimize", "Maximize", "Close"), resizing areas, and can't be dragged (unless you define a [custom draggable area](https://www.electronjs.org/docs/latest/tutorial/window-customization#set-custom-draggable-region) within the web app), can't be dropped in [Workspaces](../../../glue42-concepts/windows/workspaces/overview/index.html) or stuck to other Glue42 Windows. Frameless windows can be saved and restored in [Layouts](../../../glue42-concepts/windows/layouts/overview/index.html) and can use all Glue42 functionalities provided by the Glue42 libraries.

The following demonstrates a basic frameless window configuration:

```json
{
    "name":"frameless-app",
    "icon": "https://example.com/icon.ico",
    "type": "window",
    "details":{
        "url":"https://localhost:3000",
        "mode": "frameless",
        "width": 400,
        "height": 400
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"window"`. The `"mode"` property must be set to `"frameless"`. The `"url"` property points to the location of the web application.

### Workspaces App

A [Workspaces App](../../../glue42-concepts/windows/workspaces/overview/index.html#workspaces_concepts-frame) is a web app that can host Glue42 [Workspaces](../../../glue42-concepts/windows/workspaces/overview/index.html#workspaces_concepts-workspace).

*Note that [**Glue42 Enterprise**](https://glue42.com/enterprise/) expects only one application definition for a Workspaces App - i.e., one configuration file with `"type"` property set to `"workspaces"`. If multiple Workspaces App definitions are present, the first available one will be used.*

[**Glue42 Enterprise**](https://glue42.com/enterprise/) comes with a Workspaces UI app and a configuration file for it named `workspaces.json` and located in `%LocalAppData%\Tick42\GlueDesktop\config\apps`. If you are creating your [custom Workspaces App](../../../glue42-concepts/windows/workspaces/overview/index.html#extending_workspaces), make sure to modify or replace this file with your own configuration file, or delete it, if your application configurations are stored at another location.

The `"type"` property must be set to `"workspaces"`:

```json
{
    "title": "Workspaces UI",
    "type": "workspaces",
    "name": "workspaces-demo",
    "icon": "http://localhost:22080/resources/icons/workspaces.ico",
    "details": {
        "layouts": [],
        "url": "http://localhost:3000"
    },
    "customProperties": {}
}
```

The `"type"` and `"name"` top-level properties are required and `"type"` must be set to `"workspaces"`. The `"url"` and `"layouts"` properties in the `"details"` object are optional.

Use `"url"` to specify where the application is hosted, otherwise it will default to the Workspaces App template distributed with [**Glue42 Enterprise**](https://glue42.com/enterprise/).

#### Defining Workspace Layouts

Use the `"layouts"` property to predefine [Workspace layouts](../../../glue42-concepts/windows/workspaces/overview/index.html#workspaces_concepts-workspace_layout) that will be loaded automatically when the Workspace App starts:

```json
{
    "details": {
        "layouts": [
            // Standard Workspace layout definition.
            {
                "children": [
                    {
                        "type": "column",
                        "children": [
                            {
                                "type": "group",
                                "children": [
                                    {
                                        "type": "window",
                                        "appName": "clientlist"
                                    }
                                ]
                            },
                            {
                                "type": "group",
                                "children": [
                                    {
                                        "type": "window",
                                        "appName": "clientportfolio"
                                    }
                                ]
                            }
                        ]
                    }
                ]
            },

            // Or a simpler Workspace layout definition for an already existing layout.
            {
                "layoutName": "My Workspace"
            }
        ]
    }
}
```

Use the `"config"` property of the standard Workspace layout object or the `"restoreConfig"` property of the simpler object to pass context to the Workspace or to hide/show its tab header:

```json
{
    "details": {
        "layouts": [
            // Standard Workspace layout definition.
            {
                "children": [],
                "config": {
                    "noTabHeader": true,
                    "context": { "glue" : 42 }
                }
            },

            // Or a simpler Workspace layout definition for an already existing layout.
            {
                "layoutName": "My Workspace",
                "restoreConfig": {
                    "noTabHeader": true,
                    "context": { "glue" : 42 }
                }
            }
        ]
    }
}
```

Hiding the Workspace tab header with the `"noTabHeader"` property prevents the user from manipulating the Workspace through the UI and allows for the Workspace to be controlled entirely through API calls. For instance, a Workspace may be tied programmatically to certain logic, a button, etc., designed to manage its state without any user interaction.

### Web Group App

A Web Group App is a web app used in [**Glue42 Enterprise**](https://glue42.com/enterprise/) for handling Glue42 [web groups](../../../glue42-concepts/windows/window-management/overview/index.html#window_groups-web_groups).

*By default, [**Glue42 Enterprise**](https://glue42.com/enterprise/) will search for а registered app with the name `"glue42-web-group-application"` and if one is available, will use it as the Web Group App. If no such app is found, the first available app definition of type `"webGroup"` will be used. Note that [**Glue42 Enterprise**](https://glue42.com/enterprise/) expects only one app definition for a Web Group App - i.e., one configuration file with `"type"` property set to `"webGroup"`. If multiple Web Group App definitions are present, the first one will be used.*

[**Glue42 Enterprise**](https://glue42.com/enterprise/) comes with a Web Group App and a configuration file for it named `webGroup.json` and located in `%LocalAppData%\Tick42\GlueDesktop\config\apps`. If you are creating your [custom Web Group App](../../../glue42-concepts/windows/window-management/overview/index.html#extending_web_groups), make sure to modify or replace this file with your own configuration file, or delete it, if your app configurations are stored at another location.

The `"type"` property must be set to `"webGroup"`:

```json
{
    "name": "web-group-app",
    "title": "Web Group App",
    "type": "webGroup",
    "hidden": true,
    "details": {
        "url": "https://localhost:3000/",
        "autoOpenDevTools": true,
        "preloadScripts": ["https://example.com/my-script.js"],
        "pool": {
            "min": 5
        }
    }
}
```

The `"url"` property is required and must point to the location of your custom Web Group App.

Use the `"autoOpenDevTools"` property to automatically open the Chrome Dev Tools (disabled by default) when debugging your Web Group App.

Use the `"preloadScripts"` property to specify a list of URLs pointing to scripts that will be loaded and executed before loading the Web Group App.

Use the `"pool"` property to specify the minimum number of cached Web Group App instances (default is `3`) used for improving group performance and user experience. The higher the number, the more memory will be consumed; the lower the number, the higher the chance to experience delay during web group operations.

The `"hidden"` property is set to `true` in order to hide the Web Group App from the Glue42 [Toolbar](../../../glue42-concepts/glue42-toolbar/index.html), because this is a service app used directly by [**Glue42 Enterprise**](https://glue42.com/enterprise/) to handle Glue42 Window groups.

## Grouping Applications

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.10">

Applications can be grouped in folders and subfolders in the Application Manager via configuration:

![App Grouping](../../../images/toolbar/app-grouping.gif)

To group applications in folders/subfolders, use the `"customProperties"` top-level key in the application configuration file:

```json
{
    "name":"ClientList",
    "title":"Client List",
    "icon": "http://localhost:22080/resources/icons/tick42.ico",
    "type": "window",
    "customProperties": {
        "folder": "Clients"
    },
    "details":{
         "url":"http://localhost:3000/client-list"
    }
}
```

The configuration below shows how to group applications in a subfolder:

```json
{
    "customProperties": {
        "folder": "Clients/Corporate"
    }
}
```

## Default Application Bounds

When an application is started from the [Glue42 Toolbar](../../../glue42-concepts/glue42-toolbar/index.html), its initial size and position is defined in the application configuration. When the user moves or resizes an application, [**Glue42 Enterprise**](https://glue42.com/enterprise/) remembers its last position and size and uses them as initial bounds the next time the application starts. Saving the last window bounds is enabled by default for all applications, but can be disabled per application using the `"ignoreSavedLayout"` property in its configuration:

```json
{
    "name": "my-app",
    "type": "window",
    "ignoreSavedLayout": true,
    "details": {
        "url": "https://example.com"
    }
}
```

## Configuration Validator

A free [Glue42 Configuration Validator](https://marketplace.visualstudio.com/items?itemName=Tick42.glue42-configuration-validator) tool is available and you can install it as a Visual Studio Code extension. The validator tool has several functionalities:

- Performs real time validation of JSON files on save or change of the active editor.
- Can generate template configurations for different types of Glue42 applications - `"window"`, `"exe"`, `"node"` and Service Windows.
- Can deploy (copy) the created configuration to a specified application configuration folder.

*For more detailed information, see the [README](https://github.com/Tick42/vscode-glue42-app-config-validator/blob/master/README.md) file of the Glue42 Configuration Validator.*