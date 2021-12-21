## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.9">

The Intents API is accessible through the [`glue.intents`](../../../reference/glue/latest/intents/index.html) object.

*See the JavaScript [Intents example](https://github.com/Glue42/js-examples/tree/master/intents) on GitHub.*

## Finding Intents

To find all registered Intents, use the [`all()`](../../../reference/glue/latest/intents/index.html#API-all) method:

```javascript
const allIntents = await glue.intents.all();
```

To get a collection of all Intents that fit certain criteria, use the [`find()`](../../../reference/glue/latest/intents/index.html#API-find) method:

```javascript
const intents = await glue.intents.find("ShowChart");
```

The [`find()`](../../../reference/glue/latest/intents/index.html#API-find) method accepts a string or an [`IntentFilter`](../../../reference/glue/latest/intents/index.html#IntentFilter) object as an optional argument. The [`IntentFilter`](../../../reference/glue/latest/intents/index.html#IntentFilter) has the following optional properties:

| Property | Description |
|----------|-------------|
| `name` | Name of the Intent for which to search. |
| `contextType` | Context type (pre-defined data structure - e.g., `"Instrument"`) with which the Intent handler works. |

If no filter is supplied, [`find()`](../../../reference/glue/latest/intents/index.html#API-find) returns all registered Intents.

## Raising Intents

To raise an Intent, use the [`raise()`](../../../reference/glue/latest/intents/index.html#API-raise) method:

```javascript
await glue.intents.raise("ShowChart");
```

The [`raise()`](../../../reference/glue/latest/intents/index.html#API-raise) method accepts an Intent name as a string or an [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object as a required argument. The only required property of the [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object is [`intent`](../../../reference/glue/latest/intents/index.html#IntentRequest-intent) which must specify the name of the Intent to be raised.

## Targeting Intent Handlers

When raising an Intent, optionally target one or more Intent handlers using the [`target`](../../../reference/glue/latest/intents/index.html#IntentRequest-target) property of the [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object:

```javascript
const intent = await glue.intents.find("ShowChart")[0];
const intentHandler = intent.handlers[0];

const intentRequest = {
    intent: "ShowChart",
    target: { app: intentHandler.applicationName }
}

await glue.intents.raise(intentRequest);
```

The [`target`](../../../reference/glue/latest/intents/index.html#IntentRequest-target) property of the [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object accepts the following values:

| Value | Description |
|-------|-------------|
| `"startNew"` | Will start a new instance of the first available Intent handler. |
| `"reuse"` | Will reuse the first available running instance of an Intent handler or will fall back to `"startNew"` if there are no running instances available. |
| `{ app?: string, instance?: string}` | An object with optional `app` and `instance` properties. The `app` property accepts an application name, the `instance` property - an ID of a running application instance. Provide a value for the `app` property to start a new instance of a specific Intent handler application. The application name is available in the `applicationName` property of the [`IntentHandler`](../../../reference/glue/latest/intents/index.html#IntentHandler) object. Provide a value for the `instance` property to reuse a specific running instance of an Intent handler. The ID of an Intent handler instance is available in the `instanceId` property of the [`IntentHandler`](../../../reference/glue/latest/intents/index.html#IntentHandler) object. Using this targeting option gives you full control over the choice of an appropriate Intent handler. |

The default value for the `target` property is `"startNew"` when the Intent has been defined in an application configuration. If the Intent has been [registered dynamically](#registering_intents_at_runtime), the default value is `"reuse"`.

The [`IntentHandler`](../../../reference/glue/latest/intents/index.html#IntentHandler) object has a `type` property which shows whether the Intent handler is an application that will be started (`type: "app"`), or an already running instance of an Intent handler (`type: "instance"`).

*Note that in order for the running Intent handler instance to be registered as type `"instance"`, the application must use the [`addIntentListener()`](../../../reference/glue/latest/intents/index.html#API-addIntentListener) method in its code to handle context updates (see [Handling Context Updates](#context-handling_context_updates)) or to register an Intent at runtime (see [Registering Intents at Runtime](#registering_intents_at_runtime)). Otherwise, the running Intent handler instance will be of type `"app"`.*

## Context

### Passing Initial Context

To pass initial context to the Intent handler, use the [`context`](../../../reference/glue/latest/intents/index.html#IntentRequest-context) property of the [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object. It accepts an [`IntentContext`](../../../reference/glue/latest/intents/index.html#IntentContext) object as a value:

```javascript
const intentRequest = {
    intent: "ShowChart",
    target: "startNew"
    context: {
        type: "Instrument",
        data: {
            // Context for the started application.
            RIC: "MSFT"
        }
    },
    // Specify application start options for the Intent handler.
    options: {
        width: 300,
        height: 200
    }
}

await glue.intents.raise(intentRequest);
```

The [`type`](../../../reference/glue/latest/intents/index.html#IntentContext-type) property of the [`IntentContext`](../../../reference/glue/latest/intents/index.html#IntentContext) object is required and specifies the structure of the context object. The [`data`](../../../reference/glue/latest/intents/index.html#IntentContext-data) property is the actual data to be passed to the Intent handler.

The [`options`](../../../reference/glue/latest/intents/index.html#IntentRequest-options) property of the [`IntentRequest`](../../../reference/glue/latest/intents/index.html#IntentRequest) object is used to pass custom [`ApplicationStartOptions`](../../../reference/glue/latest/appmanager/index.html#ApplicationStartOptions) to the Intent handler.

### Handling Context Updates

To handle the context data passed when an Intent is raised and targeted at your application, use the [`addIntentListener()`](../../../reference/glue/latest/intents/index.html#API-addIntentListener) method. It has two required parameters - an Intent name and a context handler definition:

```javascript
// Context handler definition.
function contextHandler (context) {
    // Check the context type.
    const contextType = context.type;

    if (contextType === "Instrument") {
        // Extract the context data.
        const data = context.data;
        // Аpplication specific logic for handling the new context data.
    };
};

glue.intents.addIntentListener("ShowChart", contextHandler);
```

## Registering Intents at Runtime

To register an Intent at runtime, use the [`addIntentListener()`](../../../reference/glue/latest/intents/index.html#API-addIntentListener) method. Besides an Intent name, this method also accepts an object describing an Intent as a first required parameter:

```javascript
// Intent definition.
const intent = {
    intent: "ShowChart",
    contextTypes: ["Instrument"],
    displayName: "Fidessa Instrument Chart",
    icon: "https://example.com/resources/icon.ico"
};

// Context handler.
function contextHandler (context) {
    // Check the context type.
    const contextType = context.type;

    if (contextType === "Instrument") {
        // Extract the context data.
        const data = context.data;
        // Аpplication specific logic for handling the new context data.
    };
};

glue.intents.addIntentListener(intent, contextHandler);
```

*Note that when you register an Intent only at runtime (the Intent isn't defined in an application configuration), your application must be running in order to handle the Intent and it will always be of type `"instance"`. If your application isn't running when this Intent is raised, it won't be available as a possible Intent handler.*

## Reference

For a complete list of the available Intents API methods and properties, see the [Intents API Reference Documentation](../../../reference/glue/latest/intents/index.html).