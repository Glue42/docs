## Overview

The Layouts API allows you to save the arrangement and context of any set of applications running in [**Glue42 Enterprise**](https://glue42.com/enterprise/) as a named Layout and later restore it. You can also choose a default Global Layout which [**Glue42 Enterprise**](https://glue42.com/enterprise/) will load upon startup.

The Layouts library supports different types of Layouts:

- **Global**

This type of Layout can contain floating [Glue42 Windows](../../window-management/overview/index.html), [Glue42 Window Groups](../../window-management/javascript/index.html#window-groups), [Workspaces](../../workspaces/overview/index.html), [Activities](../../../data-sharing-between-apps/activities/overview/index.html). A Global Layout describes the bounds and context of all components participating in it.

- **Application Default**

The default Layout of an application instance describes the last saved window bounds, the window state (maximized, minimized, normal), whether the window is collapsed and the default window context.

*Note that the `"ignoreSavedLayout"` property in the [application configuration](../../../../developers/configuration/application/index.html) can be used to ignore the last saved application Layout. In this case, when you restart the application, it will use its default bounds, state and context set in the application configuration file.*

- **Activity**

The Layout of an [Activity](../../../data-sharing-between-apps/activities/overview/index.html) instance describes the arrangement of all windows participating in the Activity, as well as the context of the Activity owner window.

- **Workspace**

The Layout of a [Workspace](../../workspaces/overview/index.html) instance describes the arrangement of the Workspace elements, its bounds and the context of the applications participating in it.

## Default Global Layout

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

A default Global Layout is an already saved arrangement of Glue42 enabled applications that is restored upon startup of [**Glue42 Enterprise**](https://glue42.com/enterprise/). Often users need the same set of starting applications when fulfilling daily routines. Setting up a default Global Layout will save them the time and effort they usually spend in finding, starting and arranging the required applications on system start/restart.

In the example below, you can see how the user first creates and then saves and restores a Layout. After that, the user sets the saved Layout as the default Global Layout which is restored upon restart of [**Glue42 Enterprise**](https://glue42.com/enterprise/):

![Layouts](../../../../images/layouts/layouts.gif)

## Layout Stores

[**Glue42 Enterprise**](https://glue42.com/enterprise/) can load Layouts from a local store or from a remote REST service.

### Local Path Layout Store

By default, the Layouts are saved to and loaded from a local Layouts store, located in the `%LocalAppData%\Tick42\UserData\T42-DEMO\layouts` folder where you can store, customize and delete your Layout files locally.

### REST Service Layout Store

Layout definitions can also be hosted on a server and obtained from a REST service. 

For a reference implementation of a remote Layout definitions store, see the [Node.js REST Config](https://github.com/Tick42/rest-config-example-node-js) example. This basic implementation stores the user Layouts in files (they have the same structure as the local Layouts files) and returns the same set of data for all users (does not take the user into account). New Layouts are stored in files (using the name of the Layout - there is no validation whether the Layouts name can be used as a file name). The remove Layout operation is not implemented and just logs to the console. For instructions on running the sample server on your machine, see the `README.md` file in the repository.

For a .NET implementation of a remote Layout definitions store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

If your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy is not configured to load Layouts from a REST service, you need to edit the `system.json` file located in the `%LocalAppData%\Tick42\GlueDesktop\config` folder.

1. Locate the `layouts` top-level property:

```json
 "layouts": {
    "store": {
        "type": "file"
    }
  }
```

2. Update the Layouts store to:

```json
 "layouts": {
    "store": {
        "type": "rest",
        "restURL": "http://localhost:8004/",
        "restFetchInterval": 20
      }
  } 
```

| Property | Description |
|----------|-------------|
| `"type"` | Can be `"file"` or `"rest"`, depending on the type of Layouts store you want to setup. |
| `"restURL"` | The URL address of the Layouts REST service. |
| `"restFetchInterval"` | Interval (in seconds) for fetching Layouts from the REST service. | 