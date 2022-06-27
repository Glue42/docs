## Overview

The Layouts API allows you to save the arrangement and context of any set of apps running in [**Glue42 Enterprise**](https://glue42.com/enterprise/) as a named Layout and later restore it. You can also choose a default Global Layout which [**Glue42 Enterprise**](https://glue42.com/enterprise/) will load upon startup.

The Layouts library supports different types of Layouts:

- **Global**

This type of Layout can contain floating [Glue42 Windows](../../window-management/overview/index.html), [Glue42 Window Groups](../../window-management/javascript/index.html#window-groups), [Workspaces](../../workspaces/overview/index.html). A Global Layout describes the bounds and context of all components participating in it.

- **App Default**

The default Layout of an app instance describes the last saved window bounds, the window state (maximized, minimized, normal), whether the window is collapsed and the default window context.

- **Workspace**

The Layout of a [Workspace](../../workspaces/overview/index.html) instance describes the arrangement of the Workspace elements, its bounds and the context of the apps participating in it.

## Default Global Layout

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

A default Global Layout is an already saved arrangement of Glue42 enabled apps that is restored upon startup of [**Glue42 Enterprise**](https://glue42.com/enterprise/). Often users need the same set of starting apps when fulfilling daily routines. Setting up a default Global Layout will save them the time and effort they usually spend in finding, starting and arranging the required apps on system start or restart.

In the example below, you can see how the user first creates and then saves and restores a Layout. After that, the user sets the saved Layout as the default Global Layout which is restored upon restart of [**Glue42 Enterprise**](https://glue42.com/enterprise/):

![Layouts](../../../../images/layouts/layouts.gif)

## Bypassing App Default Layouts

The App Default Layout contains information about:

- the last saved window bounds - size and location;
- the window state - maximized, minimized or normal and whether it's collapsed;
- the default window context;

When an app is started for the first time by [**Glue42 Enterprise**](https://glue42.com/enterprise/), the size and the location of the app window are determined by the bounds set in the [app configuration](../../../../developers/configuration/application/index.html) file (or by the default bounds, if none are specified in the app configuration). When the user moves or resizes the app window and closes it, the new bounds are automatically saved as an App Default Layout and the next time the app is started, its window will be loaded using these bounds.

Sometimes, it may be necessary to bypass the App Default Layout - e.g., if somehow the app window has been saved outside the visible monitor area, or you simply want your app to always start with certain bounds, state or context despite the user interactions.

To bypass the App Default Layout only once, press and hold the `SHIFT` key and click on the app in the [Glue42 Toolbar](../../../glue42-toolbar/index.html) to start it.

To instruct [**Glue42 Enterprise**](https://glue42.com/enterprise/) to always ignore the App Default Layout for your app, use the `"ignoreSavedLayout"` top-level key in the [app configuration](../../../../developers/configuration/application/index.html) file:

```json
{
    "ignoreSavedLayout": true
}
```

## Layout Stores

[**Glue42 Enterprise**](https://glue42.com/enterprise/) can obtain Layouts from a local store, from a remote REST service, or from a [Glue42 Server](../../../glue42-server/index.html).

In the standard [**Glue42 Enterprise**](https://glue42.com/enterprise/) deployment model, Layouts aren't stored locally on the user machine, but are served remotely. If [**Glue42 Enterprise**](https://glue42.com/enterprise/) is configured to use a remote Layout store, it will poll it periodically and discover new Layouts. The store implementation is usually connected to an entitlement system based on which different users can have different Layouts.

The settings for the Layout stores are defined in the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config`, under the `"layouts"` top-level key, which accepts an object with a `"store"` property as a value.

The `"store"` object has the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `"type"` | `string` | Type of the Layout store. Can be one of `"file"` (default), `"rest"` or `"server"`. |
| `"restURL"` | `string` | URL pointing to the Layout store. Valid only in `"rest"` mode. |
| `"restFetchInterval"` | `number` | Interval in seconds at which to poll the REST service for updates. Default is `60`. Valid only in `"rest"` mode. |
| `"restClientAuth"` | `string` | Authentication configuration. Can be one of `"no-auth"` (default), `"negotiate"` or `"kerberos"`. Valid only in `"rest"` mode. |
| `"rejectUnauthorized"` | `boolean` | If `true` (default), SSL validation will be enabled for the REST server. Valid only in `"rest"` mode. |

### Local

By default, the Layouts are saved to and loaded from a local Layout store located in the `%LocalAppData%\Tick42\UserData\T42-DEMO\layouts` folder, where you can store, customize and delete your Layout files locally.

To instruct [**Glue42 Enterprise**](https://glue42.com/enterprise/) to manage Layouts as local files, set the `"type"` property of the `"store"` object to `"file"`:

```json
{
    "layouts" : {
        "store": {
            "type": "file"
        }
    }
}
```

### Remote

Layout definitions can also be obtained from remote Layout stores via a REST service.

For a reference implementation of a remote Layout definitions store, see the [Node.js REST Config](https://github.com/Glue42/rest-config-example-node-js) example. The user Layouts are stored in files with the same structure as local Layout files. This basic implementation doesn't take the user into account and returns the same set of data for all users. New Layouts are stored in files using the name of the Layout and there isn't validation for the name. The operation for removing a Layout isn't implemented and just logs to the console. For instructions on running the sample server on your machine, see the README file in the repository.

For a .NET implementation of a remote Layout definitions store, see the [.NET REST Config](https://github.com/Tick42/rest-config-example-net) example.

To configure a connection to the REST service providing the Layout store, set the `"type"` property of the `"store"` object to `"rest"`:

```json
{
    "layouts": {
        "store": {
            "type": "rest",
            "restURL": "http://localhost:8004/",
            "restFetchInterval": 20,
            "restClientAuth": "no-auth"
        }
    }
}
```

The remote store must return Layout definitions in the following response shape:

```json
{
    "layouts": [
        // List of Layout definition objects.
        {}, {}
    ]
}
```

### Glue42 Server

You can also use the [Glue42 Server](../../../glue42-server/index.html) for hosting and retrieving Layout stores. The [Glue42 Server](../../../glue42-server/index.html) is a complete server-side solution for providing data to Glue42. To configure [**Glue42 Enterprise**](https://glue42.com/enterprise/) to fetch Layouts from a [Glue42 Server](../../../glue42-server/index.html), set the `"type"` property of the `"store"` to `"server"`:

```json
{
    "layouts" : {
        "store": {
            "type": "server"
        }
    }
}
```

*Note that when using the [Glue42 Server](../../../glue42-server/index.html) as a Layout store, Layout files aren't only fetched from the server, but are also saved on the server (e.g., when the user edits and saves an existing Layout).*