## Overview

The Glue42 Toolbar is an application which acts as a central hub for managing Glue42 enabled applications, Layouts, Workspaces, search, notifications and various [**Glue42 Enterprise**](https://glue42.com/enterprise/) settings. [**Glue42 Enterprise**](https://glue42.com/enterprise/) comes with two options for a toolbar - a [Floating Toolbar](#floating_toolbar) and a [Launchpad](#launchpad). Both toolbars have certain advantages and disadvantages and are meant to present you with a choice depending on your specific requirements and preferences.

## Floating Toolbar

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

The Glue42 Floating Toolbar is always visible, can be minimized or dragged to any position on the screen. The [**Glue42 Enterprise**](https://glue42.com/enterprise/) trial edition is configured to start with this toolbar. If you prefer using a launch bar in the style of the Windows Start menu, which hides when it loses focus, see the Glue42 [Launchpad](#launchpad).

The Floating Toolbar is an entirely customizable [**Glue42 Enterprise**](https://glue42.com/enterprise/) application that can be implemented to suit your demands in regard to managing the applications, Workspaces and various [**Glue42 Enterprise**](https://glue42.com/enterprise/) settings available to the user. The toolbar UI and functionalities can be designed in accordance with specific user needs and requirements.

In the example below, you can see several helpful functionalities of the Floating Toolbar:

- moving the Floating Toolbar, switching from vertical to horizontal view and vice versa;
- switching the theme from the Settings;
- listing applications from application stores;
- using applications, choosing favorite applications;
- [Workspace](../windows/workspaces/overview/index.html) and [Layout](../windows/layouts/overview/index.html) save and restore options;
- feedback form;
- notifications button with a notifications counter;
- shutdown button for [**Glue42 Enterprise**](https://glue42.com/enterprise/);

![Toolbar](../../images/toolbar/floating-toolbar.gif)

## Launchpad

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

### Usage

The Glue42 Launchpad is a Windows style toolbar that is hidden on startup of [**Glue42 Enterprise**](https://glue42.com/enterprise/) and the user can open it via a keyboard shortcut whenever necessary. The Launchpad is non-intrusive, as it shows up on the screen only when the user needs it, and doesn't need to be moved or minimized manually. If you prefer using a free-floating toolbar that is always visible and can easily be moved to any location on the screen by dragging, see the Glue42 [Floating Toolbar](#floating_toolbar).

The Launchpad is especially beneficial for power users who feel comfortable working mainly with keyboard shortcuts. Use the `TAB` and the arrow keys to navigate through the sections and their items. Select an item with the `ENTER` key, or by clicking on it, and the Launchpad will close automatically. You can also press `ESC` or click anywhere outside the Launchpad to close it without selecting an item.

In the example below, you can see some of the main functionalities of the Launchpad:

- activating the Launchpad through a configurable shortcut;
- using the search bar to find applications or Layouts;
- starting applications;
- saving, restoring and removing Layouts;
- marking favorite items;
- pinning the Launchpad so it stays always on screen;
- using the Launchpad Menu from where you can restart or shutdown [**Glue42 Enterprise**](https://glue42.com/enterprise/), save Layouts, see system info, send feedback and more;

![Launchpad](../../images/toolbar/launchpad.gif)

### Configuration

The [**Glue42 Enterprise**](https://glue42.com/enterprise/) trial edition is configured to use the [Floating Toolbar](#floating_toolbar) by default. To use the Glue42 Launchpad instead, you have to modify the global system configuration from the `system.json` file of [**Glue42 Enterprise**](https://glue42.com/enterprise/) located in `%LocalAppData%\Tick42\GlueDesktop\config` and also the Launchpad configuration file, named `launchpad.json` and located in `%LocalAppData%\Tick42\GlueDesktop\config\apps`.

In the `system.json` file, set the `"useEmbeddedShell"` top-level key to `false`:

```json
{
    ...
    "useEmbeddedShell": false,
    ...
}
```

In the `launchpad.json` file, set the `"disabled"` top-level key to `false`:

```json
[
    {
        "disabled": false,
        ...
    }
]
```

Restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) for the changes to take effect. To use the Floating Toolbar again, reverse the configuration changes and restart [**Glue42 Enterprise**](https://glue42.com/enterprise/).

The default shortcut for the Launchpad is `SUPER + \` (Windows key + backslash). To change it, edit the `"shortcut"` top-level key in the `launchpad.json` file:

```json
{
    ...
    "shortcut": "ctrl+alt+space",
    ...
}
```

The users can resize the Launchpad by dragging its borders and its new bounds will be saved when [**Glue42 Enterprise**](https://glue42.com/enterprise/) shuts down or restarts. If you want the Launchpad to always start with the bounds specified in the `launchpad.json` file, set the `"ignoreSaveOnClose"` top-level key to `true`:

```json
{
    ...
    "ignoreSaveOnClose": true,
    ...
}
```

The [Placement](../windows/window-management/javascript/index.html#window_operations-placement) of the Launchpad by default is set to the bottom left corner of the screen. To change it, modify the `"placement"` property of the `"details"` top-level key. The following example demonstrates how to place the Launchpad at the top left corner of the screen and how to set its width to 30% of the screen width and its height to 40% of the screen height:

```json
{
    ...
    "details": {
        ...
        "placement": {
            "snapped": true,
            "verticalAlignment": "top",
            "horizontalAlignment": "left",
            "width": "30%",
            "height": "40%"
        }
    }
}
```

## Custom Toolbar

The source code for the Floating Toolbar is [available on GitHub](https://github.com/Glue42/toolbar). You can use it as a template for creating your own custom toolbar.

The Launchpad is available as a React Component on `npm` - [`@glue42/launchpad-ui-react`](https://www.npmjs.com/package/@glue42/launchpad-ui-react). You can use the component in your own apps and also modify it by passing different options. For more details, see the README file of the [`@glue42/launchpad-ui-react`](https://www.npmjs.com/package/@glue42/launchpad-ui-react) package.

Once you have implemented a toolbar, you must host it, create an [application configuration](../../developers/configuration/application/index.html#application_configuration) file for it and add it to your application store. Make sure to set the `"shell"` top-level key to `true`:

```json
{
    ...
    "shell": true,
    ...
}
```

Also, modify the [system configuration](../../developers/configuration/system/index.html) of [**Glue42 Enterprise**](https://glue42.com/enterprise/) from the `system.json` file - set the `"useEmbeddedShell"` property to `false`:

```json
{
    ...
    "useEmbeddedShell": false,
    ...
}
```

Restart [**Glue42 Enterprise**](https://glue42.com/enterprise/) for the changes to take effect.