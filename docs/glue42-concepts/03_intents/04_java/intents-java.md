## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

The Intents API is accessible through `glue.intents()`.

## Finding Intents

To find all registered intents, use the `all()` method:

```java
glue.intents().all();
```

## Raising Intents

To raise an Intent, use the `raise()` method:

```java
glue.intents().raise("ShowChart");
```

The `raise()` method accepts an Intent name as a `String` or an `IntentRequest` instance as an argument.

### Targeting Intent Handlers

When raising an Intent, optionally target one or more `IntentHandler` instances using the `withTarget()` method of the `IntentRequest` builder:

```java
Intent intent = glue.intents().all()
        .toCompletableFuture().join()
        .get("ShowChart");

IntentHandler intentHandler = intent.getHandlers().get(0);

IntentRequest intentRequest =
        IntentRequest.intent(intent.getName())
                .withTarget(IntentTarget.application(intentHandler.getApplicationName()))
                .build();

glue.intents().raise(intentRequest);
```

The `IntentTarget` can be created using one of the following factory methods:

| Method | Accepts | Description |
|--------|---------|-------------|
| `startNew()` | `-` | Will start a new instance of the first available Intent handler. |
| `reuse()` | `-` | Will reuse the first available running instance of an Intent handler or fallback to `startNew()` if there are no running instances available. |
| `application()` | `String` | Will start a new instance of a specific Intent handler app. |
| `instance()` | `String` | Will reuse a specific running instance of an Intent handler. |

The default value for the Intent request target is `startNew()` when an Intent handler app is available. If the Intent has only been [registered dynamically](#registering_intents_at_runtime), the default value is `reuse()`.

## Context

### Passing Initial Context

To pass initial context to the Intent handler, use the `withContext()` method of the `IntentRequest` builder. Pass the context type as a first `String` argument and the context data as a second generic type argument:

```java
Map<String, Object> contextData =
        Collections.singletonMap("data",
                Collections.singletonMap("RIC", "MSFT"));

Map<String, Object> options = new LinkedHashMap<>();
options.put("width", 300);
options.put("height", 200);

glue.intents().raise(
        IntentRequest.intent("ShowChart")
                .withTarget(IntentTarget.startNew())
                .withContext(
                        // Context type.
                        "Instrument",
                        contextData)
                .withOptions(options)
                .build()
);
```

The `withOptions()` method of the `IntentRequest` builder is used to pass custom app startup options to the Intent handler. It accepts a `Map<String,Object>` value as an argument.

### Handling Context Updates

To handle the context data passed when an Intent is raised and targeted at your app, use the `addIntentListener()` method. Pass the intent name as a first `String` argument and an `IntentContextHandler` as a second:

```java
glue.intents().addIntentListener("ShowChart", context -> {
    context.getType().ifPresent(contextType -> {
        switch (contextType) {
            case "Instrument": {
                Map<String, Object> data = context.getData();
                // App-specific logic for handling the new context data.
                break;
            }
        }
    });
    // Optionally return a result to the caller.
    return Collections.emptyMap();
});
```
## Registering Intents at Runtime

To register an Intent at runtime, use `addIntentListener()` method. Pass an `IntentListenerRequest` as a first argument and an `IntentContextHandler` as a second:

```java
IntentListenerRequest<Map<String, Object>> intent =
        IntentListenerRequest.intent("ShowChart", ReifiedType.OBJECT_MAP)
                .withContextTypes(Collections.singletonList("Instrument"))
                .withDisplayName("Instrument Chart")
                .build();

glue.intents().addIntentListener(intent, (context) -> {
    context.getType().ifPresent(contextType -> {
        switch (contextType) {
            case "Instrument": {
                Map<String, Object> data = context.getData();
                // App-specific logic for handling the new context data.
                break;
            }
        }
    });
    // Optionally return a result to the caller.
    return Collections.emptyMap();
});
```

*Note that when you register an Intent only at runtime (the Intent isn't defined in the app configuration file), your app must be running in order to handle the Intent. If your app isn't running when this Intent is raised, it won't be available as a possible Intent handler.*