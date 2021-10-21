## Retrieving Context Data

The Shared Contexts API is accessible through the [`glue.contexts`](../../../../reference/glue/latest/shared%20contexts/index.html) object.

*See the JavaScript [Shared Contexts example](https://github.com/Glue42/js-examples/tree/master/shared-context-example) on GitHub.*

To get the names of all currently available shared contexts, use the [`all()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-all) method:

```javascript
// Returns a string array with the available context names.
const availableContexts = glue.contexts.all();
```

To get the value of a specific context object, use the [`get()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-get) method:

```javascript
const data = await glue.contexts.get("app-styling");
```

## Updating a Context

Use the [`update()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-update) method to create a new shared context or update the properties of an existing shared context. New properties (context keys) will be added, existing ones will be updated, and you can also remove context keys by setting them to `null`.

```javascript
const contextUpdate = {
    backgroundColor: "red",
    alternativeColor: "green"
};

await glue.contexts.update("app-styling", contextUpdate);
```

To remove keys, send a context update and set them to `null`:

```javascript
const keysToRemove = { alternativeColor: null };

await glue.contexts.update("app-styling", keysToRemove);
```

## Replacing a Context

Other than updating a context, you have the option to replace its value completely by using the [`set()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-set) method:

```javascript
const newContext = { backgroundColor: "purple" };

// This will completely overwrite the existing context value.
await glue.contexts.set("app-styling", newContext);
```

The [`set()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-set) method overwrites the existing context object, as opposed to the [`update()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-update) method, which only updates the values of its properties.

## Subscribing for Context Updates

To subscribe for context updates, use the [`subscribe()`](../../../../reference/glue/latest/shared%20contexts/index.html#API-subscribe) method. It accepts the name of the context as a first required parameter and a function that will handle the context updates as a second required parameter:

```javascript
const handler = (context, delta, removed) => {
    const bgColor = context.backgroundColor;

    console.log(bgColor);
});

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