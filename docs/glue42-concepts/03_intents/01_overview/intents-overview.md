## Overview

In certain workflow scenarios, your app may need to start (or activate) a specific app. For instance, you may have an app showing client portfolios with financial instruments. When the user clicks on an instrument, you want to start an app which shows a chart for that instrument. In other cases, you may want to present the user with several options for executing an action or handling data from the current app.

The Intents API makes all that possible by enabling apps to register, find and raise Intents.

The case with the "Portfolio" and the "Chart" app above can be implemented in the following way:

1. The "Chart" apps registers an Intent called "ShowChart", specifying the data type (predefined data structure) that it works with - e.g., "Instrument".

2. When the user clicks on on instrument, the "Portfolio" app raises the "ShowChart" Intent, optionally specifying an Intent target, data type and app start up options.

This way, the "Portfolio" and "Chart" apps can be completely decoupled. If later the "Chart" app needs to be replaced, the new app for showing charts only needs to register the same Intent in order to replace the old one (provided that it works with the "Instrument" data structure as well).

Another case where the Intents API can be useful is if you want to find (and possibly filter) all apps that have registered a certain Intent. This may be because you want to present the user with all available (or appropriate) options for executing an action or handling data - e.g., on hover over an instrument or when clicking an instrument, the user sees a menu with all apps that have registered the Intent "ShowChart" *and* can work with the "Instrument" data structure:

1. All apps that can visualize data in charts register an Intent called "ShowChart", specifying the data structure they work with. Some of them work with "Instrument" data type, others work with different data types.

2. When the user clicks on an instrument in the "Portfolio" app, the "Portfolio" app searches for all registered Intents with a name "ShowChart" and filters them by the data type they work with.

3. The user sees a menu built on the fly which shows all currently available apps for visualizing charts that work with "Instrument" data type.

## Defining Intents

Intents are either defined through the app configuration file, or registered dynamically at runtime. User defined app files are usually located in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., `T42-DEMO`). Intents are configured under the `intents` top-level key of the app configuration file.

It is possible for different apps to register an Intent with the same name, which is useful when several apps perform the same action or work with the same data structure. This allows for easy replacement of apps. You may have an old app that has registered an Intent called `"ShowChart"` which you want to replace with a new app. Your new app only needs to register the same Intent (you can either remove the old app or leave it as an additional option for the users who prefer it). No changes to the calling app are necessary - when it raises the `"ShowChart"` Intent, the new app will be called.

Use the `intents` top-level key in the app configuration file to define an Intent:

```json
{
    "intents": [
        {
            "name": "ShowChart",
            "displayName": "BBG Instrument Chart",
            "contexts": ["Instrument"]
        }
    ]
}
```

| Property | Description |
|----------|-------------|
| `name` | **Required**. The name of the Intent. |
| `displayName` | The human readable name of the Intent. Can be used in context menus, etc., to visualize the Intent. |
| `contexts` | The type of predefined data structures with which the app can work. |