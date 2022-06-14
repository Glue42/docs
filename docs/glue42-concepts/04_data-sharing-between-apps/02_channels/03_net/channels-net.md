## Adding Channels to Your Application

### Adding the Channel Selector

For .NET applications, you can add the Channel Selector at runtime:

```csharp
IGlueWindows glueWindows = glue.GlueWindows;
var glueWindowOptions = glueWindows?.GetStartupOptions() ?? new GlueWindowOptions();

// Request Channel support, so the window will show the Channel Selector.
glueWindowOptions.WithChannelSupport(true);

// Any WPF window.
System.Windows.Window window = this;

// See also Window Management for more details on registering windows as Glue42 Windows.
glue.GlueWindows.RegisterWindow(window, glueWindowOptions)
    .ContinueWith(r =>
    {
        IGlueWindow glueWindow = r.Result;

        // See Channel subscriptions and handling data.
        glueWindow.ChannelContext.Subscribe(new LambdaGlueChannelEventHandler<T42Contact>(
            (context, channel, updatedContact) =>
            {
                T42Contact contact = updatedContact;
                HandleContact(contact);
            }, (context, newChannel, prevChannel) =>
            {
                // Handle Channel changes for this window.
                T42Contact contact = context.GetValue<T42Contact>("contact");
                HandleContact(contact);
            }), "contact");
    });
```

The callback is invoked when:
- The data from the Channel you are currently on is updated.

The second callback (Channel change handle) is invoked when:
- You switch the Channel and the application (window) is assigned to a new Channel.
- Your app (window) isn't joined to a Channel anymore (e.g. you have deselected the current Channel). In this case, it will be undefined/null.

### Channel Discovery

Your app can discover Channels in 3 ways:

- As a snapshot of the currently known Channels:

```csharp
// This will return an array of all currently known Channels.
IGlueChannel[] channels = glue.Channels.GetChannels();
```

- Through a subscription:

```csharp
// The subscription lambda will be invoked for all current Channels
// and for any newly created Channel.
glue.Channels.Subscribe(channel => HandleChannel(channel));
```

- Awaiting a Channel by name as a `Task`, or awaiting a specific Channel with a lambda filter:

```csharp
// By name.
glue.Channels.AwaitChannel("Green").ContinueWith(...);

// With a lambda filter - allows finer filtering.
glue.Channels.AwaitChannel(channel => IsChannelInteresting(channel)).ContinueWith(...);
```

### Joining a Channel

Once it finds a Channel, your app can join it:

```csharp
IGlueChannelContext glueChannelContext = glue.Channels.JoinChannel(channel);
```

The `glueChannelContext` can then be used for reading, writing and subscribing for Channel data.

## Subscribing for Data

To track the data in the current Channel, use the `subscribe()` method:

### Untyped Subscriptions

The event handler will be invoked on each Channel update. The delta items contain the changed keys and values:

```csharp
ChannelContext.Subscribe(new LambdaGlueChannelEventHandler(
    (context, channel, update) =>
    {
        DeltaItem[] added = update.Added;
        DeltaItem[] deltas = update.Deltas;
        DeltaItem[] removed = update.Removed;
    }, null), "contact");
```

### Typed Subscriptions

An alternative to the untyped subscriptions are the typed subscriptions which ease the update handling:

```csharp
ChannelContext.Subscribe(new LambdaGlueChannelEventHandler<string>(
    (context, channel, updatedValue) =>
    {
        // Updated value is of the requested type: string.
        string newFirstName = updatedValue;
        Console.WriteLine($"Name is {newFirstName}");
    }, null), "contact.name.firstName");
```

Or you can subscribe for the entire contact:

```csharp
ChannelContext.Subscribe(new LambdaGlueChannelEventHandler<T42Contact>(
    (context, channel, updatedContact) =>
    {
        // Updated value is of the requested type: T42Contact.
        T42Contact contact = updatedContact;
        HandleContact(contact);
    }, null), "contact");
```

## Reading Data from a Channel

- Reading fields:

Let's read the contact's first and last name:

```csharp
Console.WriteLine($"Contact is: {channelContext.GetValue<string>("contact.name.firstName")} {channelContext.GetValue("contact.name.lastName")}");
```

This demonstrates how to read string values from `contact.name.firstName` and `contact.name.LastName`.

- Reading types:

As an alternative to reading value by value, you can define a type:

```csharp
// Excerpt from T42 Entities.
public class T42Contact
{
    public T42Id[] Ids { get; set; }

    public string DisplayName { get; set; }

    public T42Name Name { get; set; }

    // The rest of the properties are removed for conciseness.
}

public class T42Id
{
    public string SystemName { get; set; }
    public string NativeId { get; set; }
}

public class T42Name
{
    public string CompanyName { get; set; }
    public string LastName { get; set; }
    public string FirstName { get; set; }
    public string[] OtherNames { get; set; }
    public string Honorific { get; set; }
    public string[] PostNominalLetters { get; set; }
}

```

And read the `T42Contact` object directly from its location ("contact"):

```csharp
if (channelContext.GetValue<T42Contact>("contact") is T42Contact contact)
{
    HandleContact(contact);
}
```

## Publishing Data

You can use the `SetValue()` method to update the field of a context object:

```csharp
glueChannelContext.SetValue("Jordan", "contact.name.firstName");
```

Or to update the entire object:

```csharp
    T42Contact contact = ...;
    glueChannelContext.SetValue(contact, "contact");
```