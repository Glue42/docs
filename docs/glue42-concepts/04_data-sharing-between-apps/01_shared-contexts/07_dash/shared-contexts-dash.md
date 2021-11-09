## Context Components

The Glue42 Dash library offers two components for handling shared context objects - `Context` and `Contexts`. 

The `Context` component binds to a specific context object and automatically subscribes for changes to it. You can set and update its value and also react to context updates. The `id` of the instantiated `Context` component is the name of the specific context object.

The `Contexts` component exposes an API for handling context objects by name - set and update their values, destroy contexts or set the values of nested properties using their paths in the context object.

To use the Glue42 Shared Contexts functionalities, instantiate the `Context` and/or `Contexts` components and pass an `id` to each component:

```python
import dash
import dash_glue42

app = dash.Dash(__name__)

app.layout = dash_glue42.Glue42(id="glue42", children=[
    # Using the general API to set/update contexts.
    dash_glue42.Contexts(id="g42-shared-contexts"),

    # Subscribe for a specific context.
    dash_glue42.Context(id="app-styling")
])
```

*See the Dash [Shared Contexts example](https://github.com/Glue42/glue-dash-example/tree/master/contexts) on GitHub.*

## Updating a Context

To update a shared context object, define an application callback. Pass the context component ID and its `update` property in the `Output` of the callback. For `Input` of the callback pass the ID and the property of the component that you want to trigger the context update.

The following examples demonstrate how to update the `"app-styling"` context using the `Contexts` and `Context` components. For the purpose of the examples, the `Input` component is a text input.

When using the `Contexts` component, it is required to pass the name of the context object you want to update:

```python
# Define a callback that will trigger a context update
# when a specified property of a component changes.
@app.callback(
    Output("g42-shared-contexts", "update"),
    Input("background-color", "value"),
    prevent_initial_call=True
)
def update_app_styling_context(background_color):

    return {
        "name": "app-styling",
        "data": {
            "backgroundColor": background_color
        }
    }
```

When using the `Context` component to update a specific context object, return only the update data:

```python
@app.callback(
    Output("app-styling", "update"),
    Input("background-color", "value"),
    prevent_initial_call=True
)
def update_app_styling_context(background_color):
    return { "backgroundColor": background_color }
```

Updating a context merges the existing properties of the context object with the properties of the update object. Only the specified context properties are updated, any other existing context properties will remain intact.

## Replacing a Context

To replace a shared context object, define an application callback. Pass the context component ID and its `set` property in the `Output` of the callback. For `Input` of the callback pass the ID and the property of the component that you want to trigger replacing the context.

The following examples demonstrates how to set an entirely new value for the `"app-styling"` context object using the `Contexts` and `Context` components.

When using the `Contexts` component, it is required to pass the name of the context object whose value you want to set:

```python
# Define a callback that will trigger replacing the context
# when a specified property of a component changes.
@app.callback(
    Output("g42-shared-contexts", "set"),
    Input("default-styling-btn", "n_clicks"),
    prevent_initial_call=True
)
def set_default_app_styling(n_clicks):

    return {
        "name": "app-styling",
        "data": {
            "backgroundColor": "white",
            "color": "black"
        }
    }
```

When using the `Context` component to set the value of a specific context object, return only the data:

```python
@app.callback(
    Output("app-styling", "set"),
    Input("default-styling-btn", "n_clicks"),
    prevent_initial_call=True
)
def set_default_app_styling(_):

    return {
        "backgroundColor": "white",
        "color": "black"
    }
```

Setting the value of a context removes and replaces all existing properties of the specified context object with the new ones.

## Handling Context Updates

Use the `Context` component to handle context updates. Define an application callback handler and for `Input` pass the name of the context object and its `context` property which holds the current context value. The handler will be triggered each time the context is updated. For `Output` of the callback pass the ID and the property of the component you want to update with the data from the context object.

The following example demonstrates how to update the `style` property of the `"app-wrapper"` component with data from the `"app-styling"` context object:

```python
# Define a callback that will handle updates of the context object.
@app.callback(
    Output("app-wrapper", "style"),
    Input("app-styling", "context")
)
def app_styling_context_changed_handler(context):
    
    if context is not None:
        data = context.get("data", {})
        background_color = data.get("backgroundColor")
        font_color = data.get("color")

        return {
            "backgroundColor": background_color,
            "color": font_color
        }
```

## Destroying a Context

Use the `Contexts` component to destroy a context object. For `Input` of the callback pass the ID and the property of the component you want to trigger destroying the context. For `Output`, pass the ID of the `Contexts` component and its `destroy` property. Return an object with the name of the context you want to destroy:

```python
# Define a callback that will destroy the context object.
@app.callback(
    Output("g42-shared-contexts", "destroy"),
    Input("destroy-context-btn", "n_clicks"),
    prevent_initial_call=True
)
def destroy_app_styling_context(n_clicks):
    return { "name": "app-styling" }
```