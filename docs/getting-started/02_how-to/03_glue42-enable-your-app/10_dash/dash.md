## Overview

The [Glue42 Dash](https://pypi.org/project/dash-glue42/) library offers components based on React that enable you to use Glue42 features in your apps built with the [Dash](https://dash.plotly.com/) framework for Python.

*Note that all Glue42 Dash components aren't represented visually and will be invisible in your apps. Their only purpose is to provide access to Glue42 functionalities.*

## Installation

The [Glue42 Dash](https://pypi.org/project/dash-glue42/) library is available as a package on the Python Package Index. To install it, execute the following command:

```cmd
pip install dash_glue42
```

## Referencing

To reference the Glue42 Dash library in your Dash app:

```python
import dash_glue42
```

## Initialization

To initialize the Glue42 Dash library, use the `Glue42` component and pass an `id` for it:

```python
import dash
import dash_glue42

app = dash.Dash(__name__)

app.layout = dash_glue42.Glue42(id="glue42")
```

The `Glue42` component has the following properties:

| Property | Description |
|----------|-------------|
| `id` | The ID of this component. Used to identify Dash components in callbacks. The ID must be unique across all components in an app. |
| `isEnterprise` | Indicates whether the application is running in [**Glue42 Enterprise**](https://glue42.com/enterprise/) or [**Glue42 Core**](https://glue42.com/core/). |
| `children` | The children of this component. |
| `settings` | An object containing configurations for the respective Glue42 libraries. |
| `fallback` | An optional component to display while initializing Glue42. |

The underlying Glue42 JavaScript library accepts an optional settings object used for configuring the library features. The example below demonstrates how to enable the Glue42 [Channels API](../../../../glue42-concepts/data-sharing-between-apps/channels/dash/index.html) by using the `settings` property when the app is meant to run in [**Glue42 Enterprise**](https://glue42.com/enterprise/):

```python
import dash
import dash_glue42

glue_settings = {
    # Enabling Channels in Glue42 Enterprise.
    "desktop": {
        "config": {
            "channels": True
        }
    }
}

app = dash.Dash(__name__)

# Initializing the Glue42 library with custom settings.
app.layout = dash_glue42.Glue42(id="glue42", settings=glue_settings, children=[
    # Instantiating the Channels component.
    dash_glue42.Channels(id="glue42-channels")
])
```

The table below describes the properties of the optional settings object:

| Property | Description |
|----------|-------------|
| `type` | *Optional*. Accepts either `"platform"` or `"client"` as a value. Specifies whether this is a [Main application](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html) or a [Web Client](https://core-docs.glue42.com/developers/core-concepts/web-client/overview/index.html) in the context of [**Glue42 Core**](https://glue42.com/core/). The default is `"client"`. |
| `web` | *Optional*. An object with one property: `config`. The `config` property accepts a [configuration object](https://core-docs.glue42.com/reference/core/latest/glue42%20web/index.html#Config) for the [Glue42 Web](https://www.npmjs.com/package/@glue42/web) library. You should define this object if your application is a [Web Client](https://core-docs.glue42.com/developers/core-concepts/web-client/overview/index.html). |
| `webPlatform` | *Optional*. An object with one property: `config`. The `config` property accepts a [configuration object](https://core-docs.glue42.com/developers/core-concepts/web-platform/setup/index.html#configuration) for the [Web Platform](https://www.npmjs.com/package/@glue42/web-platform) library. You should define this object if your application is a [Main application](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html) in the context of [**Glue42 Core**](https://glue42.com/core/). |
| `desktop` | *Optional*. An object with one property: `config`. The `config` property accepts a [configuration object](../../../../reference/glue/latest/glue/index.html#Config) for the [@glue42/desktop](https://www.npmjs.com/package/@glue42/desktop) library used in [**Glue42 Enterprise**](https://glue42.com/enterprise/). You should define this object if your application is a [**Glue42 Enterprise**](https://glue42.com/enterprise/) application. |

*Note that in Python it isn't possible to pass a function as a value, therefore the properties of the `config` objects which accept a function as a value can't be set.*

## Application Configuration

To add your Dash app to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with application configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`).

*Note that this step isn't necessary if your application is running in a [**Glue42 Core**](https://glue42.com/core/) project. For more details, see the [Application Definitions](https://core-docs.glue42.com/capabilities/application-management/index.html#application_definitions) section in the [**Glue42 Core**](https://glue42.com/core/) documentation.*

The following is an example configuration for a Dash app:

```json
{
    "title": "Dash App",
    "type": "window",
    "name": "dash-app",
    "details": {
        "url": "http://127.0.0.1:5000/dash-app",
        "top": 25,
        "left": 800,
        "height": 400,
        "width": 400
    }
}
```

*For more details, see the [Application Configuration](../../../../developers/configuration/application/index.html) section.*

*See also the [Glue42 Dash examples](https://github.com/Glue42/glue-dash-example) on GitHub, demonstrating the various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

## Glue42 Dash Concepts

For more detailed information on the different Glue42 concepts and APIs, see:

- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/dash/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/dash/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/dash/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/dash/index.html)
- [Notifications](../../../../glue42-concepts/notifications/dash/index.html)