## Listing Applications

The .NET **Application Management** API is accessible through `glue.AppManager`.

To get a collection of all available Glue42 enabled applications, use the `AwaitApplications()` method:

```csharp
var applications = await glue.AppManager.AwaitApplications();
```

To get a specific application, use `AwaitApplication()` and pass a predicate function for finding the desired application:

```csharp
var app = await glue.AppManager.AwaitApplication(a => a.Name == "appName");
```

*See the .NET [Application Management example](https://github.com/Tick42/net-examples/tree/master/app-management-demo) on GitHub.*

## Starting Applications

To start an application, use the `Start()` method of an application instance:

```csharp
// Get the application.
var app = await glue.AppManager.AwaitApplication(a => a.Name == "appName");

// Start the application.
app.Start();
```

### Application Context

The `Start()` method accepts a context object (application starting context) as an optional parameter. The following example demonstrates how to create an application context and pass it to the application you want to start:

```csharp
// Create application context.
var context = AppManagerContext.CreateNew();
context.Set("selectedClient", "3");

// Pass the context when starting the application.
app.Start(context);
```

## Applications Instances

Each Glue42 enabled application may have multiple running instances. Each instance of a .NET application may have multiple windows belonging to that particular instance.

### Current Instance

To get a reference to the current instance of your application, use the `MyInstance` property:

```csharp
var myAppInstance = glue.AppManager.MyInstance;
```

### Instance Windows

To get a collection of all Glue42 Windows belonging to an application instance, use the `Windows` property of the application instance:

```csharp
// Get an application.
var app = await AppManager.AwaitApplication(a => a.Name == "appName");
// Get the windows belonging to the application instance.
var appWindows = app.Instances.FirstOrDefault()?.Windows;
```

## Events

The .NET Application Management API offers methods for monitoring application and instance events - adding/removing applications, starting/stopping instances and more.

### Application

To get notified when an application definition has been added, use the `ApplicationAdded` event:

```csharp
glue.AppManager.ApplicationAdded += (appManager, appArgs) =>
    {
        var appName = appArgs.Application.Name;
    };
```

To get notified when an application definition has been removed, use the `ApplicationRemoved` event:

```csharp
glue.AppManager.ApplicationRemoved += (appManager, appArgs) =>
    {
        var appName = appArgs.Application.Name;
    };
```

To get notified when an application definition has been updated, use the `ApplicationUpdated` event:

```csharp
glue.AppManager.ApplicationUpdated += (appManager, appArgs) =>
    {
        var appType = appArgs.Applicaton.ApplicationType;
    };
```

### Instance

To get notified when an application instance has been started, use the `ApplicationInstanceStarted` event:

```csharp
glue.AppManager.ApplicationInstanceStarted += (appManager, instanceArgs) =>
    {
        var instanceId = instanceArgs.Instance.Id;
    };
```

To get notified when an application instance has been stopped, use the `ApplicationInstanceStopped` event:

```csharp
glue.AppManager.ApplicationInstanceStopped += (appManager, instanceArgs) =>
    {
        var instanceId = instanceArgs.Instance.Id;
    };
```

## Multi Window Apps

Glue42 .NET offers support for applications (WPF and WinForms) consisting of multiple windows. Each window can be registered not only as a [Glue42 Window](../../windows/window-management/net/index.html#glue42_windows), but also as a Glue42 application that you can save and restore in a [Layout](../../windows/layouts/overview/index.html), start directly from the Glue42 Toolbar, etc. This features allows you to register WPF and WinForms windows at runtime and to properly persist their state when saving and restoring a Layout. 

*See the .NET [Multi Window Demo example](https://github.com/Tick42/net-examples/tree/master/multiwindow-demo) on GitHub.*

### Registering Apps at Runtime

To register a WPF window as an application at runtime, use the `RegisterWPFApp()` generic method. It accepts the types of the window, the state to save/restore and the execution context that can be passed to the application and provides an application definition builder:

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

- `Shutdown()` - Invoked when the application is closed by [**Glue42 Enterprise**](https://glue42.com/enterprise/).

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

Once the windows of the app are registered as Glue42 applications, the application becomes responsible for announcing when initiating a new instance through its logic in order to notify [**Glue42 Enterprise**](https://glue42.com/enterprise/) about the new instance. If the window has been started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) (e.g., from the Glue42 Toolbar), this is handled automatically. Use the `RegisterAppWindow()` from the [Window Management API](../../windows/window-management/net/index.html#registering_windows_as_app_instances) method or a combination of the `RegisterWindow()` method (see [Registering Windows](../../windows/window-management/net/index.html#registering_windows)) and the `RegisterInstance()` methods.

The `RegisterAppWindow()` method registers the window both as a Glue42 Window and as a Glue42 application instance. It accepts the WPF window object (or the WinForms window handle), the Glue42 application object, the application name and an application options builder as arguments:

```csharp
var placement = new GlueWindowScreenPlacement();

glue.GlueWindows.RegisterAppWindow(myWindow, myWindow, myWindowAppName,
        // Specify application options.
        builder => builder
        .WithPlacement(placement)
        .WithType(GlueWindowType.Tab));
```

The `RegisterWindow()` method accepts the WPF window object (or the WinForms window handle) and `RegisterInstance()` accepts the application name, the Glue42 Window ID, the Glue42 application object and the synchronization context as arguments:

```csharp
// Register the window as a Glue42 Window.
glue.GlueWindows.RegisterWindow(myWindow).ContinueWith(r =>
    {
        // Register the Glue42 application instance.
        glue.AppManager.RegisterInstance(myWindowAppName, r.Result.Id, myWindow, SynchronizationContext.Current);
    });
```

### Saving and Restoring State

To save the state of applications registered at runtime, use the `SetSaveRestoreStateEndpoint()` method of the `InitializeOptions` class to provide application state when initializing the Glue42 library in your main window:

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