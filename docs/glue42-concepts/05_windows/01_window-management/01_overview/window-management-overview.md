## Overview

The Window Management API lets you create and manipulate windows and is the basis of the [Application Management](../../../application-management/overview/index.html) API. It allows users to group a set of desktop windows so that they move, maximize and minimize together. The Window Management API provides the following features, not found in any normal browser:

- 3 types of window modes: flat, tab or HTML
- Complete control and customization of the windows:
	- visibility - create hidden windows, show them later;
	- bounds - set window location and/or size;
	- user interaction - allow a window to be sticky, enable/disable minimize/maximize/close buttons;
	- add custom buttons to the windows and respond accordingly to user interaction with them;
	- organize windows into tabs that the user can also tear off;

Native applications, as opposed to web applications, can have more than one window. This means that after you make your native application Glue42 enabled, your application windows don't automatically become Glue42 enabled. You can choose which windows to register (or not register) as Glue42 windows so that they can use [**Glue42 Enterprise**](https://glue42.com/enterprise/) functionalities.

Web and native windows are handled by [**Glue42 Enterprise**](https://glue42.com/enterprise/) as window abstractions. This means that:
- You can use any technology adapter we offer (JavaScript, .NET, Java, etc.) to control any window (web or native).
- From an end-user perspective, there is no difference between web or native windows.
- Feature parity is provided by the different technology adapters.

## Window Modes

Glue42 supports three different window modes: flat, tab and HTML. The window mode is controlled by the `"mode"` window setting, which can be specified in the application configuration or as a setting when opening a Glue42 Window programmatically.

Application configuration settings:

```javascript
{
    "title": "Client List",
    "type": "window",
    "name": "clientlist",
    "details": {
        "url": "http://localhost:22080/index.html",
        "mode": "html"
    }
}
```

### Flat Windows

Flat windows are available for web and native apps. 

![Image of a flat window](../../../../images/window-management/window-mode-flat.png)

### Tab windows

Tab windows are available for web and native apps.

![Image of a tab window](../../../../images/window-management/window-mode-tab.png)

### HTML Windows

HTML windows are available for web apps only.

![Image of an HTML window](../../../../images/window-management/window-mode-html.png) 