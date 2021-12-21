## Global Layouts

Global saving and restoring is an operation in which all applications running on a user's desktop are saved to a named layout which can later be restored.

### Saving a Global Layout

The **Layouts** API can be accessed through `glue.Layouts`.

To save a global layout, use `glue.Layouts.Save()` passing a configuration object with a required `name` property. Note that if a layout with that `name` already exists, it will be replaced. The resolved `Task` includes the saved layout information, which can then be used or manipulated.

```csharp
glue.Layouts.Save(new SaveOptions
{
    Name = name
}).ContinueWith(t =>
{
    ILayout savedLayout = t.Result;
    // handle saved layout
});
```

### Restoring a Global Layout

To restore a global layout, use `glue.Layouts.Restore()` passing the global layout object.

```csharp
glue.Layouts.Restore(layout, new RestoreOptions
{
    RestoreMode = CurrentInstances.CloseAndReuse
}).ContinueWith(t =>
{
    ILayout restoredLayout;
    // handle restoredLayout
});
```

## Managing Layouts

### Listing Layouts

If you want a snapshot of all layouts currently available, you can use:

```csharp
glue.Layouts.List() // returns an array of layout objects
```

### Removing Layouts

To remove a layout, use the `glue.Layouts.Remove()` method. You must pass the layout instance to be removed:

```csharp
glue.Layouts.Remove(layout).ContinueWith(t => 
{
    ILayout removedLayout = t.Result;
    // handle removed layout
});
```

### Layout Events

Your application can react to layout events, such as adding, removing, updating or renaming a layout:

```csharp
// notifies when a new layout is added
glue.Layouts.LayoutAdded += LayoutsManagerOnLayoutEvent;

// notifies when a layout is removed
glue.Layouts.LayoutRemoved += LayoutsManagerOnLayoutEvent;

// notifies when a layout is changed
glue.Layouts.LayoutChanged += LayoutsManagerOnLayoutEvent;

// notifies when a layout is renamed
glue.Layouts.LayoutRenamed += LayoutsManagerOnLayoutEvent;

private void LayoutsManagerOnLayoutEvent(object sender, LayoutEventArgs e)
{
    switch (e.LayoutEvent)
    {
        case LayoutEvent.OnLayoutAdded:
            HandleLayoutEvent(() => OnLayoutAdded(e.Layout));
            break;
        case LayoutEvent.OnLayoutChanged:
            HandleLayoutEvent(() => OnLayoutChanged(e.Layout));
            break;
        case LayoutEvent.OnLayoutRemoved:
            HandleLayoutEvent(() => OnLayoutRemoved(e.Layout));
            break;
        case LayoutEvent.OnLayoutRenamed:
            HandleLayoutEvent(() => OnLayoutRenamed(e.Layout, e.PrevLayout));
            break;
        default:
            throw new ArgumentOutOfRangeException();
    }
}
```

## Saving Custom Data

When a global layout is saved, applications can store custom data with it. When the global layout is restored, the custom data is also restored and returned to the applications.

*Note that saving large volumes of custom data as window context (e.g., thousands of lines of table data) can lead to significant delays when saving a layout. A layout usually contains several (in some cases - many) applications and/or Workspaces (which can also contain many apps) and if one or more of the apps saves large amounts of context data each time a layout is saved, this will significantly slow down the saving process. The methods for saving custom context work best with smaller amounts of data. If your application needs to save large amounts of data, you have to think about how to design this process better - for instance, you may store IDs, indices, etc., as context data, save the actual data to a database and when you restore the layout, fetch the data using the data IDs saved as window context.*

The custom data is the **window context** - each window (application) can store window specific context. When restored, the window will have the saved context.

To save custom data, applications can subscribe for layout save requests. The callback passed as an argument will be invoked when a layout save is requested.
You can define your own typed configuration for your application:

```csharp
public class AppState
{
    public string SelectedClient { get; set; }
    public bool DarkThemeOn { get; set; }
}
```

Then when initializing Glue42 you can specify the save/restore endpoint as a typed (or untyped) callback:

```csharp
var initializeOptions = new InitializeOptions();
initializeOptions.SetSaveRestoreStateEndpoint(saveState => new AppState
    {
        DarkThemeOn = false,
        SelectedClient = "some client"
    }.AsCompletedTask(),
    restoreState =>
    {
        AppState stateToRestore = restoreState;
        // restore state
    });

Glue42.InitializeGlue(initializeOptions).ContinueWith(t => {
    Glue42 glue = t.Result;
    // handle initialized Glue42
});
```

For convenience, the save state endpoint requires a task, so you can save your state in an asynchronous fashion.

Another way of restoring your state is to fetch it from the Glue42 instance when it has been initialized:

```csharp
var initializeOptions = new InitializeOptions();
// we don't specify a restore state callback but only save state callback
initializeOptions.SetSaveRestoreStateEndpoint(saveState => new AppState
    {
        DarkThemeOn = false,
        SelectedClient = "some client"
    }.AsCompletedTask());

Glue42.InitializeGlue(initializeOptions).ContinueWith(t => 
{
    Glue42 glue = t.Result;
    
    // Glue42 is initialized, so we can extract the restore state in a typed fashion like so:
    AppState restoreState = glue.GetRestoreState<AppState>();

    // handle restoreState
});
```

If you want to return any object and work in an untyped manner, you can do it by using an `object` instead of specifying a generic `<T>` type, and using the property `OnSaveState` like so:

```csharp
Glue42.InitializeGlue(new InitializeOptions
{
    OnSaveState = _ => new Dictionary<string, object>().AsCompletedTask<object>()
}).ContinueWith(t => 
{
    Glue42 glue = t.Result;
    var restoreState = glue.GetRestoreState<Dictionary<string, object>>();
});
``` 