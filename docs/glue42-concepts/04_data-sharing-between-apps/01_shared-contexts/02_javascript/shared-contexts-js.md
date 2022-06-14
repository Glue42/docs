## Setting a Context

The Shared Contexts API is accessible through the [`glue.contexts`](../../../../reference/glue/latest/shared%20contexts/index.html) object.

*See the JavaScript [Shared Contexts example](https://github.com/Glue42/js-examples/tree/master/shared-contexts) on GitHub.*

To create a shared context or replace entirely the value of an existing one, use the [`set()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-set) method:

```javascript
const newContext = { backgroundColor: "purple" };

// This will create a new shared context, or if the context already exists,
// will completely overwrite its value.
await glue.contexts.set("app-styling", newContext);
```

The [`set()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-set) method overwrites the existing value of the shared context object, as opposed to the [`update()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-update) method, which only updates the values of its properties.

## Getting a Context

To get the value of a specific shared context, use the [`get()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-get) method:

```javascript
const data = await glue.contexts.get("app-styling");
```

## Listing All Contexts

To get the names of all currently available shared contexts, use the [`all()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-all) method:

```javascript
// Returns a string array with the available context names.
const availableContexts = glue.contexts.all();
```

## Updating a Context

To update the value of an existing shared context, use the [`update()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-update) method. New properties will be added, existing ones will be updated, and you can also remove shared context keys by setting them to `null`. If the specified shared context doesn't exist, it will be created:

```javascript
const contextUpdate = {
    backgroundColor: "red",
    alternativeColor: "green"
};

await glue.contexts.update("app-styling", contextUpdate);
```

To remove a shared context key, set it to `null`:

```javascript
const keyToRemove = { alternativeColor: null };

await glue.contexts.update("app-styling", keyToRemove);
```

## Updating Specific Properties

You can use the [`setPath()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-setPath) and [`setPaths()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-setPaths) methods to update specific shared context properties using a dot-separated string path to point to the location of the property within the shared context object. If the property (or the path) doesn't exist, it will be created. These methods are useful for updating or creating one or more nested properties within the shared context object.

To update or create a single property, use the [`setPath()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-setPath) method. It accepts the name of the shared context, a path to the property to update, and a value for the property:

```javascript
const path = "text.color";
const value = "grey";

await glue.contexts.setPath("app-styling", path, value);

// Assuming the context already exists and has this shape:
// { backgroundColor: "red" }, it will be updated as follows:
// { backgroundColor: "red", text: { color: "grey" } }
```

To update or create multiple properties, use the [`setPaths()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-setPaths) method. It accepts the name of the shared context and a list of [`PathValue`](../../../../reference/glue/latest/shared%20contexts/index.html#PathValue) objects each containing a path to the property to update and a value for it:

```javascript
const updates = [
    { path: "table.cells", value: { width: 50, height: 30 } },
    { path: "text.color", value: "white" }
];

await glue.contexts.setPaths("app-styling", updates);

// Assuming the context already exists and has this shape:
// { backgroundColor: "red", text: { color: "grey" } }, it will be updated as follows:
//
// {
//     backgroundColor: "red",
//     text: {
//         color: "white"
//     },
//     table: {
//         cells: {
//             width: 50,
//             height: 30
//         }
//     }
// }
```

## Subscribing for Context Updates

To subscribe for context updates, use the [`subscribe()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-subscribe) method. It accepts the name of the shared context as a first required parameter and a function that will handle the context updates as a second required parameter:

```javascript
const handler = (context, delta, removed) => {
    const bgColor = context.backgroundColor;

    console.log(bgColor);
};

await glue.contexts.subscribe("app-styling", handler);
```

## Unsubscribing

The [`subscribe()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-subscribe) method returns a `Promise` which resolves with a function you can use to unsubscribe from context updates:

```javascript
const unsubscribe = await glue.contexts.subscribe("app-styling", handler);

unsubscribe();
```

## Destroying a Context

To destroy a context object, use the [`destroy()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-destroy) method:

```javascript
await glue.contexts.destroy("app-styling");
```

## Reference

For a complete list of the available Shared Contexts API methods and properties, see the [Shared Contexts API Reference Documentation](../../../../reference/glue/latest/shared%20contexts/index.html).