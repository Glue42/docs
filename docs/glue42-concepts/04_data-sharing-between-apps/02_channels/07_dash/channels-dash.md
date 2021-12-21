## Adding Channels to Your App

To add the Channel Selector to your window, set `"allowChannels"` to `true` in your application configuration file under the `"details"` top-level key:

```json
{
    "title": "Dash App",
    "type": "window",
    "name": "dash-app",
    "details": {
        "url": "http://127.0.0.1:5000/dash-app",
        "mode": "tab",
        "allowChannels": true
    }
}
```

For more information on configuring your applications, see the [Application Configuration](../../../../developers/configuration/application/index.html) section.

## Channels Component

In [**Glue42 Enterprise**](https://glue42.com/enterprise/), the Channels API is disabled by default. To enable it, set the `channels` property of the configuration object to `True` when initializing the Glue42 library. Instantiate the `Channels` component and pass an `id` for it:

```python
import dash
import dash_glue42

glue_settings = {
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
    dash_glue42.Channels(id="g42-channels")
])
```

*See the Dash [Channels example](https://github.com/Glue42/glue-dash-example/tree/master/channels) on GitHub.*

## Discovering Channels

Use the `all` and `list` properties of the `Channels` component to discover all available Channels or retrieve their context data. The `all` property returns a collection of the names of all available Channels, while the `list` property returns a collection of [ChannelContext](../../../../reference/glue/latest/channels/index.html#ChannelContext) objects containing the Channel name, context data and meta data.

The following example demonstrates how to extract and display the names of all available Channels using the `list` property:

```python
def channels_contexts_to_dpd_options(channelsContexts):
    no_channel = {"label": "No Channel", "value": ""}

    if channelsContexts is not None:
        options = map(lambda channel: {
                      "label": channel.get('name'), "value": channel.get('name')}, channelsContexts)
        return [no_channel] + list(options)
    return [no_channel]

@app.callback(
    Output("channels-list", "options"),
    Input("g42-channels", "list")
)
def update_channels_list(contexts):
    return channels_contexts_to_dpd_options(contexts)
```

[**Glue42 Enterprise**](https://glue42.com/enterprise/) provides a Channel Selector which you can add to your app (see [Adding Channels to Your App](#adding_channels_to_your_app)), but if you are working on a [**Glue42 Core**](https://glue42.com/core/) project, you will have to create your own Channel Selector. 

The `all` property of the `Channels` component returns the names of all Channels and you can use it to discover and display all available Channels in the UI of your app. Define an application callback that will be triggered when the property changes in order to update the list of Channels. For `Input` of the callback pass the ID of the `Channels` component and its `all` property. For `Output` of the callback pass the component in which you want to display the Channels:

```python
@app.callback(
    Output("channels-list", "options"),
    Input("g42-channels", "all")
)
def update_channels_list(all_channels):
    return channels_to_dpd_options(all_channels)
```

The following complete example demonstrates how to discover and display all available Glue42 Channels:

```python
import dash
import dash_html_components as html
import dash_core_components as dcc
import dash_glue42

# Dropdown option that will be used to leave the current Channel.
no_channel = {"label": "No Channel", "value": ""}

app = dash.Dash(__name__)

app.layout = dash_glue42.Glue42(id="glue42", settings=glue_settings, children=[
    dash_glue42.Channels(id="g42-channels"),

    # This is an example visual representation. You can use Dash components of your choice.
    html.Div(children=[
        html.Label("Select Channel: "),
        dcc.Dropdown(id="channels-list", clearable=False)
    ]),
])


def channels_to_dpd_options(channels):

    if channels is not None:
        options = map(lambda channel: {
                      "label": channel, "value": channel}, channels)
        return [no_channel] + list(options)

    return [no_channel]


@app.callback(
    Output("channels-list", "options"),
    Input("g42-channels", "all")
)
def update_channels_list(all_channels):
    return channels_to_dpd_options(all_channels)
```

## Joining or Leaving a Channel

To join or leave a Channel, define callbacks and use the ID and the `join` and `leave` properties of the `Channels` component for `Output`. For `Input`, use the component from which the user will select the Channels.

The following example demonstrates how to handle joining and leaving Channels:

```python
# Joining a Channel.
@app.callback(
    Output("g42-channels", "join"),
    Input("channels-list", "value")
)
def join_channel(channel_name):

    if channel_name != no_channel["value"]:  
        return {
            "name": channel_name
        }

# Leaving a Channel.
@app.callback(
    Output("g42-channels", "leave"),
    Input("channels-list", "value")
)
def leave_channel(channel_name):

    if channel_name == no_channel["value"]:
        return {}
```

## Subscribing for Data

To subscribe for Channel data, use the `my` property of the `Channels` component. It holds an object representing the context of the currently selected Channel. Define a callback that will be triggered each time the context object in the `my` property is updated or the Channel is changed. For `Input` of the callback pass the ID of the `Channels` component and its `my` property. For `Output` of the callback pass the component and a property you want to update with the Channel data.

The following example demonstrates how to extract and use the updated Channel data. The first `Output` updates the background color of the `"channels-list"` component with the color of the current Channel. The second `Output` displays the current Channel data:

```python
# Handling Channel updates.
@app.callback(
    [
        Output("channels-list", "style"),
        Output("channel-data", "children")
    ],
    Input("g42-channels", "my")
)
def channel_changed(channel):

    if channel is None:
        return [None, ""]

    channel_name = channel["name"]
    color = channel.get("meta", {"color": ''})["color"]
    data = channel.get("data", {})
    time_stamp = data.get("time")

    return [
        {"backgroundColor": color},
        "Received time: {}".format(time_stamp) if (
            time_stamp is not None) else "No time currently on Channel {}".format(channel_name)
    ]
```

## Publishing Data

To publish data to the current Channel, use the `publish` property of the `Channels` component. Define a callback that will be triggered each time you want your app to publish data to the Channel - e.g., the user selects an item, clicks a button, etc. For `Input` of the callback pass the ID and a property of the component from which your app will publish data. For `Output` of the callback pass the ID of the `Channels` component and its `publish` property.

The callback must return an object with a `data` property which holds an object with the Channel context data.

The following example demonstrates how to publish data (a timestamp) to the current Channel when the user clicks a button:

```python
import time

# Publishing data to the current Channel.
@app.callback(
    Output("g42-channels", "publish"),
    Input("publish-data", "n_clicks"),
    prevent_initial_call=True
)
def publish_data(_):

    now = time.time()
    
    return {
        "data": {
            "time": now
        }
    }
```

To publish data to a Channel that your application isn't currently on, specify the name of the Channel in the returned object:

```python
import time

# Publishing data to a different Channel.
@app.callback(
    Output("g42-channels", "publish"),
    Input("publish-data", "n_clicks"),
    prevent_initial_call=True
)
def publish_data(_):

    now = time.time()
    
    return {
        "name": "Green",
        "data": {
            "time": now
        }
    }
```