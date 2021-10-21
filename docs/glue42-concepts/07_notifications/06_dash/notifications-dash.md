## Notifications Component

To be able to raise notifications from your Dash app, instantiate the `Notifications` component and assign an ID to it:

```python
import dash
import dash_glue42

app = dash.Dash(__name__)

app.layout = dash_glue42.Glue42(id="glue42", children=[
    dash_glue42.Notifications(id="g42-notifications")
])
```

*See the Dash [Notifications example](https://github.com/Glue42/glue-dash-example/tree/master/notifications) on GitHub.*

*Note that to be able to raise notifications in a [**Glue42 Core**](https://glue42.com/core/) project and use notification actions, you have to properly setup your [Main app](https://core-docs.glue42.com/developers/core-concepts/web-platform/overview/index.html). For more details, see the [Notifications](https://core-docs.glue42.com/capabilities/notifications/setup/index.html) section of the [**Glue42 Core**](https://glue42.com/core/) documentation.*

## Raising Notifications from a Web App

To raise a notification, define an application callback handler and for `Output` pass the ID and the `raise` property of the `Notifications` component. For `Input` use the component that you want to trigger raising the notification. The callback must return an object with [notification options](../../../reference/glue/latest/notifications/index.html#Glue42NotificationOptions) - title, body, actions, etc.

The following example demonstrates raising a notification with a title, body and an action button. The action is handled by an already [registered Interop method](../../data-sharing-between-apps/interop/dash/index.html#method_registration):

```python
@app.callback(
    Output("g42-notifications", "raise"),
    Input("raise-notification", "n_clicks"),
    prevent_initial_call=True
)
def raise_notification(_):
    interop_settings = { "method": "HandleNotificationClick" }

    return {
        "title": "New Trade",
        "body": "VOD.L: 23 shares sold @ $212.03",
        "actions": [
            {
                "action": "openClientPortfolio",
                "title": "Open Portfolio"
                "interop": interop_settings
            }
        ]
    }
```

![Raising Notifications](../../../images/notifications/notification.png)

### Notification Options

The [`Glue42NotificationOptions`](../../../reference/glue/latest/notifications/index.html#Glue42NotificationOptions) object extends the standard web `NotificationOptions` object with several additional properties:

| Property | Description |
|----------|-------------|
| `actions` | An array of [`Glue42NotificationAction`](../../../reference/glue/latest/notifications/index.html#Glue42NotificationAction) objects. |
| `title` | The title of the notification. |
| `clickInterop` | Accepts an [`InteropActionSettings`](../../../reference/glue/latest/notifications/index.html#InteropActionSettings) object as a value. Use this property to [invoke an Interop method](../../data-sharing-between-apps/interop/dash/index.html#method_invocation) when the user clicks on the notification. You can specify arguments for the method and an [Interop target](../../data-sharing-between-apps/interop/dash/index.html#targeting). |
| `severity` | Defines the urgency of the notification which is represented visually by different colors in the notification UI. Can be `"Low"`, `"Medium"`, `"High"`, `"Critical"` or `"None"`. |
| `source` | Overrides the source of the notification. Provide the name of the Glue42 application which you want to be displayed as a source of the notification. |
| `type` | Accepts `"Notification"` or `"Alert"` as a value. This property is meant to be used only as a way to distinguish between notification types in case you want to create different visual representations for them - e.g., the `"Notification"` type may be considered a general notification, while the `"Alert"` type may be considered a more important or urgent notification. |


## Notification Click

To handle notification clicks, use the `clickInterop` property of the [`Glue42NotificationOptions`](../../../reference/glue/latest/notifications/index.html#Glue42NotificationOptions) object and specify an Interop method that will be invoked when the user clicks on the notification:

```python
@app.callback(
    Output("g42-notifications", "raise"),
    Input("raise-notification", "n_clicks"),
    prevent_initial_call=True
)
def raise_notification(_):

    interop_settings = {
        "method": "HandleNotificationClick",
        "arguments": {
            "name": "Vernon Mullen",
            "id": "1"
        }
    }

    notification_options = {
        "title": "New Trade",
        "body": "VOD.L: 23 shares sold @ $212.03",
        "clickInterop": interop_settings
    }

    return notification_options
```

## Notification Actions

You can create action buttons for the notification. When the user clicks on an action button, the specified callbacks will be invoked. 

![Actions](../../../images/notifications/actions.png)

*Note that the action buttons in a Glue42 Notification are limited to two, as the web browsers currently support a maximum of two actions.*

To handle action button clicks, use the `interop` property of the action object and specify an Interop method that will be [invoked](../../data-sharing-between-apps/interop/dash/index.html#method_invocation) when the user clicks the action button in the notification:

```python
@app.callback(
    Output("g42-notifications", "raise"),
    Input("raise-notification", "n_clicks"),
    prevent_initial_call=True
)
def raise_notification(_):
    
    call_client_interop = {
        "method": "CallClient",
        "arguments": {
            "name": "Vernon Mullen"
        }
    }

    open_portfolio_interop = {
        "method": "OpenPortfolio",
        "arguments": {
            "name": "Vernon Mullen",
            "id": "1"
        }
    }

    notification_options = {
        "title": "New Trade",
        "body": "VOD.L: 23 shares sold @ $212.03",
        "actions": [
            {
                "action": "callClient",
                "title": "Call Client",
                "interop": call_client_interop
            },
            {
                "action": "openClientPortfolio",
                "title": "Open Portfolio"
                "interop": open_portfolio_interop
            }
        ]
    }

    return notification_options
```