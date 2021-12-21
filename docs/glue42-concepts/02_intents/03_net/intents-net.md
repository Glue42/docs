## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

The Intents API is accessible through `glue.Intents`.

## Finding Intents

To find all registered Intents, use the `GetIntents()` method:

```csharp
var intents = await glue.Intents.GetIntents();
```

To find a specific Intent, use the `AwaitIntent()` method:

```csharp
var intent = await glue.Intents.AwaitIntent("ShowChart");
```

## Raising Intents

To raise an Intent, use the `Raise()` method. It accepts an Intent name (or Intent object) and an Intent request builder as arguments:

```csharp
await glue.Intents.Raise("ShowChart",
    builder => builder.WithContext("instrument", new {id = new {ticker = "msft"}}));
```

## Targeting Intent Handlers

When raising an Intent, optionally target one or more Intent handlers using the Intent request builder to specify a handler:

```csharp
var intent = await glue.Intents.AwaitIntent("ShowChart");
var handler = intent.Handlers[0];

await glue.Intents.Raise(intent,
    // Pass the desired Intent handler.
    builder => builder.WithHandler(handler));          
```

Or:

```csharp
await glue.Intents.Raise("ShowChart",
    builder => 
        // Target a specific Intent handler.
        builder.WithTargetSelector(handler =>
            handler.ApplicationName.Contains("InstrumentChart")));
```

## Context

To pass initial context to the Intent handler:

```csharp
await glue.Intents.Raise("ShowChart",
    builder => builder.WithContext("instrument", new {id = new {ticker = "msft"}}));
```

To handle the context in the Intent handler, use the `AddIntentListener()` method (see [Registering Intents at Runtime](#registering_intents_at_runtime)):

```csharp
glue.Intents.AddIntentListener("ShowChart",
    intent => intent.WithDisplayName("Intrument Chart"),
    context =>
    {
        Dispatcher.BeginInvoke((Action) (() =>
        {
            var wnd = new Window
            {
                // Handle the passed context.
                Content = new TextBox { Text = context.Data.AsString }
            };
            wnd.Show();
        }));
    });
```

To pass [application starting context](../../application-management/net/index.html#starting_applications-application_context) to the Intent handler:

```csharp
// Create application context.
var context = AppManagerContext.CreateNew();
context.Set("instrument", "msft");

await glue.Intents.Raise("ShowChart",
    builder => builder.WithAppStartContext(context));
```

## Registering Intents at Runtime

To register an Intent at runtime, use the `AddIntentListener()` method. It accepts an Intent name, an Intent builder and an Intent context handler as arguments:

```csharp
// Register an Intent at runtime.
glue.Intents.AddIntentListener("ShowChart",
    // Build the Intent.
    intent => intent.WithDisplayName("Intrument Chart"),
    // Specify how to handle the passed context.
    context =>
    {
        Dispatcher.BeginInvoke((Action) (() =>
        {
            var wnd = new Window
            {
                // Use the context.
                Content = new TextBox { Text = context.Data.AsString }
            };
            wnd.Show();
        }));
    });
```

*Note that when you register an Intent only at runtime (the Intent isn't defined in the application configuration file), your application must be running in order to handle the Intent. If your application isn't running when this Intent is raised, it won't be available as a possible Intent handler.*