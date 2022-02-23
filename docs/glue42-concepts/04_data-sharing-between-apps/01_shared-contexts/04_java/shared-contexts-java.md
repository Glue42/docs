## Listing All Available Contexts

A shared context object is a `Map` containing cross application data. You can access all available context objects in order to manipulate them:

```java
Collection<String> names = glue.contexts().names();
```

## Subscribing for a Context

To subscribe for shared context updates, use the `glue.contexts().subscribe()` call, passing the name of the desired context object:

```java
glue.contexts().subscribe("app-styling")
                .thenAccept(context -> context.data(data -> {
            // use context data here
        }));
```

If the specified shared context object doesn't exist, it will be created. In the example above, the changes to the context object can be handled in the `context.data(data -> {})` callback.

## Updating a Context

You can also update a shared context object by using `glue.contexts().update()` and passing as a first parameter the name of the context object you want to update, and as a second parameter - a `Map<String, Object>` with the delta values:

```java
glue.contexts().update("app-styling", Collections.singletonMap("backgroundColor", "red"));
```

If the key you pass in the `Map` object exists in the shared context object, it will be overwritten with the new value. If it doesn't exist, it will be created. If the value of a key in the `Map` you pass to `glue.contexts().update()` is `null`, then that key will be deleted from the shared context object.

Let's say you have subscribed to a shared context named `"app-styling"`, which already has two entries - `"backgroundColor", "red"` and `"alternativeColor", "yellow"`. In the example below:
- `"backgroundColor"` will be overwritten with a new value - `"blue"`;
- `"alternativeColor"` will be deleted from the shared context object;
- `"borderColor"` will be created as a new key in the shared context object and its value will be set to `"grey"`;

```java
glue.contexts()
        .subscribe("app-styling")
        .thenAccept(context -> {
            context.data(data -> {
                // use context data here
            });

            Map<String, Object> delta = new HashMap<>();
            delta.put("backgroundColor", "blue");
            delta.put("alternativeColor", null);
            delta.put("borderColor", "grey");

            context.update(delta);
        });
```