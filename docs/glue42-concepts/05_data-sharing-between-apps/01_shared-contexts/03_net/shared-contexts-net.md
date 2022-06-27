## Creating & Getting a Context

The Shared Contexts API is accessible through `glue.Contexts`.

*See the .NET [Shared Contexts example](https://github.com/Glue42/net-examples/tree/master/shared-contexts) on GitHub.*

To create a shared context, use the `GetContext()` method and pass a name for the context:

```csharp
IContext context = await glue.Contexts.GetContext("MyContext");
```

If the context already exists, you will get a reference to it.

## Listing All Contexts

To get the names of all currently available shared contexts, use the `GetContexts()` method:

```csharp
string[] contextNames = glue.Contexts.GetContexts();
```

## Setting a Context

To replace entirely the value of a shared context, use its `Set()` method:

```csharp
await context.Set(new { Alpha = "A", Beta = "B" });

// You can use `Dictionary` as well.
await context.Set(new Dictionary<string, object> { { "Alpha", "A" }, { "Beta", "B" } });
```

The `Set()` method of a shared context object overwrites its existing value, as opposed to its `Update()` method, which only updates the values of its properties.

## Updating a Context

To update a shared context, get a reference to it and modify it directly by accessing its properties, or use its `Update()` method.

You can update some of the properties of the object, while the others remain the same:

```csharp
// Get or create a shared context.
IContext context = await glue.Contexts.GetContext("MyContext");

// Modify the shared context properties directly.
context["Contact"] = new { AccountName = "jsmith", Name = "John Smith" };
context["Stocks"] = new Dictionary<string, List<object>> { { "Names", new List<object> { "AAPL", "MSFT", "GOOG" } } };

// Or use the Update() method of the shared context object.
await context.Update(ctx => { ctx["Phone"] = "555-089573" });
```

To remove a property from the shared context object, set it to `null`:

```csharp
context["Alpha"] = null;
```

The following example demonstrates creating and updating a shared context in a more strongly typed manner:

```csharp
// Define user-specific context with its own properties.
public class Status
{
    public string State { get; set; }
    public DateTime Date { get; set; }
}

public interface IMyContextType : IContext
{
    Status MyProperty { get; set; }
    int SomeOtherProperty { get; set; }
}

IMyContextType myContext = await glue.Contexts.GetContext<IMyContextType>("TestContext");

myContext.MyProperty = GetCurrentStatus();
```

## Subscribing for Context Updates

To subscribe for shared context updates, use the `ContextUpdated` event of the shared context object. Add an event handler for it to react to data updates of the shared context. The handler will receive as arguments the updated shared context and an object containing information about the updated data - list of

```csharp
// Add a handler for the context updates.
context.ContextUpdated += onContextUpdated;

// Implement a handler for the context updates.
void onContextUpdated(object sender, ContextUpdatedEventArgs e)
{
    IContext context = (IContext)sender;

    // Get the context name.
    string contextName = context.ContextName;
    // Get the entire object containing the context data.
    object data = context.GetValue("data");
    // Get the value of a specific field in the context object.
    string name = context.GetValue("data.Contact.Name") as string;

    // Get all changed fields.
    DeltaItem[] deltas = e.Deltas;
    // Get only the newly added fields.
    DeltaItem[] added = e.Added;
    // Get only the removed fields.
    DeltaItem[] removed = e.Removed;
    // Get only the updated fields.
    DeltaItem[] updated = e.Updated;

    // Each `DeltaItem` contains the name of the respective context field, its before and after values,
    // the ID and the instance of the app that has modified the field,
    // and the modification command for the field - either "Set" or "Remove".

    Console.WriteLine($"Context \"{contextName}\" has been updated.");

    foreach (DeltaItem delta in deltas)
    {
        Console.WriteLine(delta.ToString());
        Console.WriteLine(
            $"Updater ID: {delta.UpdaterId}, Updater instance: {delta.Updater}, Command: {delta.Command}"
        );
    }
}
```

## Subscribing for Context Events

To subscribe for the event which fires when a shared context is created, use the `Created` event:

```csharp
glue.Contexts.Created += onContextCreated;

void onContextCreated(object sender, ContextStateEventArgs e)
{
    Console.WriteLine("Context created: " + new { e.ContextId, e.ContextName });
}
```

To subscribe for the event which fires when a shared context is destroyed, use the `Removed` event:

```csharp
glue.Contexts.Removed += onContextRemoved;

void onContextRemoved(object sender, ContextStateEventArgs e)
{
    Console.WriteLine("Context removed: " + new { e.ContextId, e.ContextName });
}
```

## Unsubscribing

To unsubscribe from shared context updates or events, remove the handler for the respective event:

```csharp
// Unsubscribe from context updates.
context.ContextUpdated -= onContextUpdated;

// Unsubscribe from context created and destroyed events.
glue.Contexts.Created -= onContextCreated;
glue.Contexts.Removed -= onContextRemoved;
```