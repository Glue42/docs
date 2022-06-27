## Method Registration

*See the Dash [Interop example](https://github.com/Glue42/glue-dash-example/tree/master/interop) on GitHub.*

To register an Interop method and make it available to other apps, instantiate the `MethodRegister` component. Pass an ID for the component, a method definition and specify whether the method returns a result or not. The method definition can be either the method name as a string or a [MethodDefinition](../../../../reference/glue/latest/interop/index.html#MethodDefinition) object with a `name` property holding the method name, method signature and other method properties. It is mandatory to specify whether the method returns a result to the calling app.

The following example demonstrates how to register two Interop methods, one of which returns a result and the other is void:

```python
import dash
import dash_glue42

app = dash.Dash(__name__)

app.layout = dash_glue42.Glue42(id="glue42", children=[
    # Registering an Interop method that returns a result.
    dash_glue42.MethodRegister(id="g42-register-sum", definition={"name": "Sum"}, returns=True),

    # Registering an Interop method that doesn't return a result.
    dash_glue42.MethodRegister(id="g42-register-send-message", definition="SendMessage", returns=False)
])
```

Define a handler callback for every registered Interop method. The handler will be triggered each time the method is invoked. For `Input` of the callback pass the ID of the respective `MethodRegister` component and its `invoke` property. If the method returns a result to the caller, for `Output` of the callback pass the ID of the respective `MethodRegister` component and its `result` property, otherwise pass the ID and a property of the component you want to update directly.

The `MethodRegister` component has an `error` property which is set in case the method registration fails. You can use it to check whether the Interop method has been registered successfully.

To set the amount of time the component should wait for a reply from the Dash backend, use the `methodResponseTimeoutMs` property:

```python
# Specifying time to wait for a reply from the method handler. The default timeout is 30000 ms.
dash_glue42.MethodRegister(id="g42-register-sum", definition={"name": "Sum"}, returns=True, methodResponseTimeoutMs=20000)
```

*Note that it is mandatory to return the assigned `invocationId` when the Interop method returns a result, otherwise the caller won't receive the result.*

The example below demonstrates how to define handlers for the previously registered Interop methods "Sum" and "SendMessage". The handler for the "Sum" method validates the input arguments and returns either their sum or an error in the `result` property of the respective `MethodRegister` component. The "SendMessage" method handler directly updates another component with the message received as an argument of the invocation:

```python
# Helper to validate the input arguments for the "Sum" method.
def is_number(s):
    try:
        float(s)
        return True
    except ValueError:
        return False

# Handler for the "Sum" Interop method.
@app.callback(
    Output("g42-register-sum", "result"),
    Input("g42-register-sum", "invoke")
)
def sum_invocation_handler(invoke):

    if invoke is None:
        raise PreventUpdate

    invocationId = invoke.get("invocationId")
    args = invoke.get("args", {})
    a = args.get("a")
    b = args.get("b")

    are_numbers = is_number(a) and is_number(b)
    if are_numbers:
        total = float(a) + float(b)

        # When a method isn't void, you must always return the assigned `invocationId`,
        # otherwise the caller won't receive the result.
        return {
            "invocationId": invocationId,
            "invocationResult": {
                "sum": total
            }
        }
    else:
        return {
            "invocationId": invocationId,
            "error": {
                "message": "The arguments must be numbers!"
            }
        }

# Handler for the "SendMessage" Interop method.
@app.callback(
    Output("message", "children"),
    Input("g42-register-send-message", "invoke")
)
def send_message_invocation_handler(invoke):

    if invoke is not None:
        args = invoke.get("args", {})
        message = args.get("message", "")
        return message
```

## Method Invocation

To invoke an Interop method registered by another app, instantiate the `MethodInvoke` component and pass an ID for it:

```python
import dash_glue42

app.layout = dash_glue42.Glue42(id="glue42", children=[
    # A component which will invoke the "Sum" Interop method.
    dash_glue42.MethodInvoke(id="g42-invoke-sum"),

    # A component which will invoke the "SendMessage" Interop method.
    dash_glue42.MethodInvoke(id="g42-invoke-send-message")
])
```

Define a callback that will trigger invocations of the Interop method. For `Output` of the callback pass the ID of the respective `MethodInvoke` component and its `invoke` property.

The following example demonstrates how to define a callback for triggering the "Sum" Interop method when the user clicks a button. The arguments for the method are taken from UI inputs:

```python
# Callback that will trigger "Sum" invocation.
@app.callback(
    Output("g42-invoke-sum", "invoke"),
    Input("sum-numbers-btn", "n_clicks"),
    State("number-a", "value"),
    State("number-b", "value"),
    prevent_initial_call=True
)
def sum_numbers(_, a, b):

    return {
        "definition": {
            "name": "Sum"
        },
        "argumentObj": {
            "a": a,
            "b": b
        }
    }
```

If the Interop method returns a result to the caller, define another callback for handling the result. For `Input` of the callback pass the ID of the respective `MethodInvoke` component and its `result` property. Consume the result as per your app logic.

The following example demonstrates how to extract the returned result and the invocation error (if any):

```python
# Callback that will handle the result returned by "Sum".
@app.callback(
    Output("sum-numbers-result", "children"),
    Input("g42-invoke-sum", "result")
)
def sum_numbers_result_handler(result):

    if result is None:
        raise PreventUpdate

    error = result.get("error")
    hasError = error is not None
    if hasError:
        return error.get("message", '')
    else:
        invocationResult = result.get("invocationResult", {})
        sumValue = invocationResult.get("returned", {}).get("sum")
        return "Sum is {}".format(sumValue)
```

Complete example of Interop method invocation:

```python
import dash
from dash.exceptions import PreventUpdate
from dash.dependencies import Input, Output, State
import dash_html_components as html
import dash_core_components as dcc
import dash_glue42
from run import server

app = dash.Dash(__name__, server=server, routes_pathname_prefix="/app-a/")

app.layout = dash_glue42.Glue42(id="glue42", children=[
    # A component which will invoke the "Sum" Interop method.
    dash_glue42.MethodInvoke(id="g42-invoke-sum"),

    # A component which will invoke the "SendMessage" Interop method.
    dash_glue42.MethodInvoke(id="g42-invoke-send-message"),

    html.Div([
        dcc.Input(id="number-a", type="text",
                  autoComplete="off", value=37),
        dcc.Input(id="number-b", type="text",
                  autoComplete="off", value=5),
        html.Button(id="sum-numbers-btn", children="Sum"),
    ]),
    html.P(id="sum-numbers-result"),

    html.Hr(),

    html.Div(
        [
            html.Label("Message: "),
            dcc.Input(id="message", type="text",  autoComplete="off",
                      value="Send your daily report!"),
            html.Button(id="send-message", children="Send")
        ]
    )
])

# Callback that will trigger "Sum" invocation.
@app.callback(
    Output("g42-invoke-sum", "invoke"),
    Input("sum-numbers-btn", "n_clicks"),
    State("number-a", "value"),
    State("number-b", "value"),
    prevent_initial_call=True
)
def sum_numbers(_, a, b):

    return {
        "definition": {
            "name": "Sum"
        },
        "argumentObj": {
            "a": a,
            "b": b
        }
    }

# Callback that will handle the result returned by "Sum".
@app.callback(
    Output("sum-numbers-result", "children"),
    Input("g42-invoke-sum", "result")
)
def sum_numbers_result_handler(result):

    if result is None:
        raise PreventUpdate

    error = result.get("error")
    hasError = error is not None
    if hasError:
        return error.get("message", '')
    else:
        invocationResult = result.get("invocationResult", {})
        sumValue = invocationResult.get("returned", {}).get("sum")
        return "Sum is {}".format(sumValue)

# Callback that will trigger "SendMessage" invocation.
@app.callback(
    Output("g42-invoke-send-message", "invoke"),
    Input("send-message", "n_clicks"),
    State("message", "value"),
    prevent_initial_call=True
)
def send_message(_, message):

    return {
        "definition": {
            "name": "SendMessage"
        },
        "argumentObj": {
            "message": message
        }
    }
```

## Targeting

When invoking an Interop method, you can target all Interop servers, the best Interop server or all except the current one. The "best" Interop server is the app instance which has registered the Interop method first. Use the `target` property of the `invoke` object to specify the desired target.

The `target` property accepts the following values:

| Value | Description |
|-------|-------------|
| `"all"` | Targets all Interop servers that have registered the method. |
| `"best"` | Default. Targets only the Interop server that has registered the method first. |
| `"skipMine"` | Targets all Interop servers except the current one. |

The example below demonstrates how to target all Interop servers that have registered the "Sum" method:

```python
@app.callback(
    Output("g42-invoke-sum", "invoke"),
    Input("sum-numbers-btn", "n_clicks"),
    State("number-a", "value"),
    State("number-b", "value"),
    prevent_initial_call=True
)
def sum_numbers(_, a, b):

    return {
        "definition": {
            "name": "Sum"
        },
        "argumentObj": {
            "a": a,
            "b": b
        },
        "target": "all"
    }
```