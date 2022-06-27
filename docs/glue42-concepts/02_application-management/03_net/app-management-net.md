## Listing Apps

The .NET App Management API is accessible through `glue.AppManager`.

To get a collection of all available Glue42 enabled apps, use the `AwaitApplications()` method:

```csharp
var apps = await glue.AppManager.AwaitApplications();
```

To get a specific app, use `AwaitApplication()` and pass a predicate function for finding the desired app:

```csharp
var app = await glue.AppManager.AwaitApplication(a => a.Name == "appName");
```

*See the .NET [App Management example](https://github.com/Glue42/net-examples/tree/master/app-management-demo) on GitHub.*

## Starting Apps

To start an app, use the `Start()` method of an app instance:

```csharp
// Get the app.
var app = await glue.AppManager.AwaitApplication(a => a.Name == "appName");

// Start the app.
app.Start();
```

### App Context

The `Start()` method accepts a context object (app starting context) as an optional parameter. The following example demonstrates how to create an app context and pass it to the app you want to start:

```csharp
// Create app context.
var context = AppManagerContext.CreateNew();
context.Set("selectedClient", "3");

// Pass the context when starting the app.
app.Start(context);
```

## App Instances

Each Glue42 enabled app may have multiple running instances. Each instance of a .NET app may have multiple windows belonging to that particular instance.

### Current Instance

To get a reference to the current instance of your app, use the `MyInstance` property:

```csharp
var myAppInstance = glue.AppManager.MyInstance;
```

### Instance Windows

To get a collection of all Glue42 Windows belonging to an app instance, use the `Windows` property of the app instance:

```csharp
// Get an app.
var app = await AppManager.AwaitApplication(a => a.Name == "appName");
// Get the windows belonging to the app instance.
var appWindows = app.Instances.FirstOrDefault()?.Windows;
```

## Events

The .NET App Management API offers methods for monitoring app and instance events - adding/removing apps, starting/stopping instances and more.

### App

To get notified when an app definition has been added, use the `ApplicationAdded` event:

```csharp
glue.AppManager.ApplicationAdded += (appManager, appArgs) =>
    {
        var appName = appArgs.Application.Name;
    };
```

To get notified when an app definition has been removed, use the `ApplicationRemoved` event:

```csharp
glue.AppManager.ApplicationRemoved += (appManager, appArgs) =>
    {
        var appName = appArgs.Application.Name;
    };
```

To get notified when an app definition has been updated, use the `ApplicationUpdated` event:

```csharp
glue.AppManager.ApplicationUpdated += (appManager, appArgs) =>
    {
        var appType = appArgs.Applicaton.ApplicationType;
    };
```

### Instance

To get notified when an app instance has been started, use the `ApplicationInstanceStarted` event:

```csharp
glue.AppManager.ApplicationInstanceStarted += (appManager, instanceArgs) =>
    {
        var instanceId = instanceArgs.Instance.Id;
    };
```

To get notified when an app instance has been stopped, use the `ApplicationInstanceStopped` event:

```csharp
glue.AppManager.ApplicationInstanceStopped += (appManager, instanceArgs) =>
    {
        var instanceId = instanceArgs.Instance.Id;
    };
```

## Multi Window Apps

Glue42 .NET offers support for apps (WPF and WinForms) consisting of multiple windows. Each window can be registered not only as a [Glue42 Window](../../windows/window-management/net/index.html#glue42_windows), but also as a Glue42 app that you can save and restore in a [Layout](../../windows/layouts/overview/index.html), start directly from the Glue42 Toolbar, etc. This features allows you to register WPF and WinForms windows at runtime and to properly persist their state when saving and restoring a Layout.

*See the .NET [Multi Window Demo example](https://github.com/Glue42/net-examples/tree/master/multiwindow-demo) on GitHub.*

### Registering Apps at Runtime

To register a WPF window as an app at runtime, use the `RegisterWPFApp()` generic method. It accepts the types of the window, the state to save/restore and the execution context that can be passed to the app and provides an app definition builder:

```csharp
glue.AppManager.RegisterWPFApp<MyWindow, MyState, MainWindow>(app =>
    {
        app.WithName(AppName)
            .WithTitle(AppTitle)
            // Passing the Main Window as context.
            .WithContext(this)
             // Registering the app as a tab window.
            .WithType(GlueWindowType.Tab);
    });
```

The registered WPF window should implement the `IGlueApp` interface where you can specify the types of the state to save/restore and the execution context. This interface has the following methods:

- `GetState()` - Invoked when [**Glue42 Enterprise**](https://glue42.com/enterprise/) requests the current state of the window (when the window is being saved in a Layout).

Implementing the `GetState()` method:

```csharp
public async Task<SymbolState> GetState()
{
    // Returning the state that will be saved when the window is saved in a Layout.
    return Dispatcher.Invoke(() =>
    {
        // `SymbolState` is the class used for the state of the window.
        var state = new SymbolState()
        {
            ActiveSymbol = Symbol.Text
        };

        return state;
    });
}
```

- `Initialize()` - Invoked to restore the state when the window is started by [**Glue42 Enterprise**](https://glue42.com/enterprise/). Accepts the execution context, the app state to restore, a `glue` object, starting context for the app and the Glue42 Window itself.

Implementing the `Initialize()` method:

```csharp
public void Initialize(MainWindow context, SymbolState state, Glue42 glue, GDStartingContext startingContext, IGlueWindow glueWindow)
{
    // Invoked when the window is restored or opened by Glue42 Enterprise.
    Dispatcher.Invoke(() =>
    {
        Symbol.Text = state?.ActiveSymbol ?? DefaultSymbol;
    });
}
```

- `Shutdown()` - Invoked when the app is closed by [**Glue42 Enterprise**](https://glue42.com/enterprise/).

Implementing the `Shutdown()` method:

```csharp
public void Shutdown()
{
    // Here you can implement your own graceful cleanup of native resources, connections, etc., for this child app.
    // After that, close the window - otherwise, Glue42 Enterprise will wait for it to timeout and then will force close it.
    Close();
}
```

*The same functionality is provided for WinForms windows through the `RegisterWinFormsApp()` method.*

### Registering App Instances

Once the windows of the app are registered as Glue42 apps, the app becomes responsible for announcing when initiating a new instance through its logic in order to notify [**Glue42 Enterprise**](https://glue42.com/enterprise/) about the new instance. If the window has been started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) (e.g., from the Glue42 Toolbar), this is handled automatically. Use the `RegisterAppWindow()` from the [Window Management API](../../windows/window-management/net/index.html#registering_windows_as_app_instances) method or a combination of the `RegisterWindow()` method (see [Registering Windows](../../windows/window-management/net/index.html#registering_windows)) and the `RegisterInstance()` methods.

The `RegisterAppWindow()` method registers the window both as a Glue42 Window and as a Glue42 app instance. It accepts the WPF window object (or the WinForms window handle), the Glue42 app object, the app name and an app options builder as arguments:

```csharp
var placement = new GlueWindowScreenPlacement();

glue.GlueWindows.RegisterAppWindow(myWindow, myWindow, myWindowAppName,
        // Specify app options.
        builder => builder
        .WithPlacement(placement)
        .WithType(GlueWindowType.Tab));
```

The `RegisterWindow()` method accepts the WPF window object (or the WinForms window handle) and `RegisterInstance()` accepts the app name, the Glue42 Window ID, the Glue42 app object and the synchronization context as arguments:

```csharp
// Register the window as a Glue42 Window.
glue.GlueWindows.RegisterWindow(myWindow).ContinueWith(r =>
    {
        // Register the Glue42 app instance.
        glue.AppManager.RegisterInstance(myWindowAppName, r.Result.Id, myWindow, SynchronizationContext.Current);
    });
```

### Saving and Restoring State

To save the state of apps registered at runtime, use the `SetSaveRestoreStateEndpoint()` method of the `InitializeOptions` class to provide app state when initializing the Glue42 library in your main window:

```csharp
var initializeOptions = new InitializeOptions();

initializeOptions.SetSaveRestoreStateEndpoint(_ =>
    // Return the state to be saved when the applicaiton is saved in a Layout.
    new AppState
    {
        SelectedIndex = Selector.SelectedIndex
    }.AsCompletedTask(), null, Dispatcher);

Task<Glue42> glue = Glue42.InitializeGlue(initializeOptions);
```

To use the state when the window is restored, use the `GetRestoreState()` method of the Glue42 API object:

```csharp
var appState = glue.GetRestoreState<AppState>();

Selector.SelectedIndex = appState?.SelectedIndex ?? -1;
```

## Bootstrapped Apps

A common use-case is to have a bootstrapping app that handles user login and launches your apps. The bootstrapper can be any app and in the general case it isn't Glue42 enabled, but the apps it launches are. This scenario can be handled both through [app configuration](#bootstrapped_apps-app_configuration) or [runtime configuration](#bootstrapped_apps-runtime_configuration) when initializing Glue42 in the respective apps.

*Note that defining app configurations for Glue42 enabled .NET apps isn't mandatory. Glue42 enabled .NET apps announce themselves to [**Glue42 Enterprise**](https://glue42.com/enterprise/) automatically when started. The configuration file is necessary only if you want your app to be launchable from [**Glue42 Enterprise**](https://glue42.com/enterprise/). If you decide to create app configurations for the bootstrapper and the bootstrapped apps, the name of the bootstrapper specified in the `"name"` property of the bootstrapper configuration, in the `"launcherApp"` property of the bootstrapped app configuration, and in the initialization options in the bootstrapped app code must be the same.*

You must specify the name of the bootstrapping app and clear the starting context passed by [**Glue42 Enterprise**](https://glue42.com/enterprise/) to it. Clearing the starting context is very important, because [**Glue42 Enterprise**](https://glue42.com/enterprise/) by default will store the command line arguments with which the bootstrapper has been started and will try to pass them again on restore, which in some cases may not be possible or may lead to undesired results.

### App Configuration

To specify the name of the bootstrapper in the [app configurations](../../../developers/configuration/application/index.html#app_configuration-exe) of the bootstrapped apps, use the `"launcherApp"` property of the `"details"` top-level key:

```json
// Configuration file for the bootstrapped app.
{
    "name": "BootstrappedApp",
    "type": "exe",
    "details": {
        "path": "%GDDIR%/PathToBootstrappedApp/",
        // Must match the app name specified in the
        // bootstrapper configuration and in the bootstrapped app code.
        "launcherApp": "MyBootstrapper"
    }
}
```

To clear the starting context for the bootstrapper, set the `"startingContextMode"` property of the `"details"` top-level key to `"none"`. The following example configuration demonstrates how to instruct [**Glue42 Enterprise**](https://glue42.com/enterprise/) to:

- prevent saving the bounds of the bootstrapper;
- exclude the bootstrapper from participating in [Layouts](../../windows/layouts/overview/index.html);
- prevent starting multiple instances of the bootstrapper;
- run the bootstrapper as a hidden window;
- clear the starting context for the bootstrapper;
- track the bootstrapper using its process, because it isn't Glue42 enabled;
- prevent closing the bootstrapper on shutdown;

```json
// Example configuration file for the bootstrapper.
{
    "title": "My Bootstrapper",
    "type": "exe",
    // Must match the name specified in the `"launcherApp"` property of the
    // bootstrapped app configuration and in the bootstrapped app code.
    "name": "MyBootstrapper",
    "ignoreSavedLayout": true,
    "ignoreFromLayouts": true,
    "allowMultiple": false,
    "hidden": true,
    "details": {
        "path": "C:/PathToBootstrapper/",
        "command": "MyBootstrapper.exe",
        "parameters": "",
        "startingContextMode": "none",
        "trackingType": "Process",
        "terminateOnShutdown": false,
    }
}
```

### Runtime Configuration

It is mandatory to specify the name of the bootstrapper in the `InitializeOptions` object during the [initialization](../../../getting-started/how-to/glue42-enable-your-app/net/index.html) of Glue42 in your bootstrapped app, irrespective of whether you have created an [app configuration](../../../developers/configuration/application/index.html#app_configuration-exe) for it. This is necessary, because when a Glue42 enabled .NET app announces itself to [**Glue42 Enterprise**](https://glue42.com/enterprise/), it overwrites the configuration settings specified in its configuration file.

The following example demonstrates how to instruct [**Glue42 Enterprise**](https://glue42.com/enterprise/) to:

- prevent closing the bootstrapped app on shutdown;
- prevent passing previously saved by [**Glue42 Enterprise**](https://glue42.com/enterprise/) startup arguments to the bootstrapped app on restore;
- exclude the bootstrapped app from participating in [Layouts](../../windows/layouts/overview/index.html);
- prevent starting multiple instances of the bootstrapped app;

```csharp
// Must match the name specified in the configuration of the bootstrapper
// and in the `"launcherApp"` property of the bootstrapped app configuration.
const string LauncherApp = "MyBootstrapper";

var initializeOptions = new InitializeOptions
    {
        // Must match the name specified in the bootstrapped app configuration, if any.
        ApplicationName = "BootstrappedApp",
        AppDefinition = new AppDefinition
        {
            Title = "My Bootstrapped App",
            TerminateOnShutdown = false,
            LauncherApp = LauncherApp,
            StartupArguments = "",
            IgnoreFromLayouts = true,
            AllowMultiple = false
        }
    };
```

Optionally, you can also register the bootstrapper at runtime. The registration is executed by the bootstrapped app, because the bootstrapper isn't Glue42 enabled:

```csharp
// Must match the name specified in the configuration of the bootstrapper
// and in the `"launcherApp"` property of the bootstrapped app configuration.
const string LauncherApp = "MyBootstrapper";

private static async Task RegisterBootstrapper(Glue42 glue, string bootstrapperLocation, string bootstrapperFile)
{
    var tcs = new TaskCompletionSource<(MethodInvocationStatus, string)>(TaskCreationOptions
        .RunContinuationsAsynchronously);

    glue.GetService<IApplicationRegistryService>().RegisterHostApplication(new AppConfig
    {
        Name = LauncherApp,
        Type = "exe",
        Details = new AppConfigDetails
        {
            // Clearing the starting context for the bootstrapper in order to prevent
            // Glue42 Enteprise from passing startup arguments to it on restore.
            StartingContextMode = StartingContextMode.None,
            // Track the bootstrapper through its process, because it isn't Glue42 enabled.
            TrackingType = TrackingType.Process,
            Command = bootstrapperFile,
            Path = bootstrapperLocation,
            // Prevent closing the bootstrapper on shutdown.
            TerminateOnShutdown = false
        },
        // Run the bootstrapper in a hidden Glue42 Window.
        Hidden = true,
        // Prevent starting multiple instances of the bootstrapper
        AllowMultiple = false,
        // Don't autostart the bootstrapper when Glue42 Entperise starts.
        AutoStart = false,
        // Exclude the bootstrapper from participating in Layouts.
        IgnoreFromLayouts = true,
        //Don't save the bootstrapper bounds.
        IgnoreSavedLayout = true
    }, AppDefinitionLifetime.Default, r => tcs.TrySetResult((r.Status, r.ResultMessage)));

    if (await tcs.Task.ConfigureAwait(false) is var status && status.Item1 != MethodInvocationStatus.Succeeded)
    {
        throw new Exception(status.Item2);
    }
}
```