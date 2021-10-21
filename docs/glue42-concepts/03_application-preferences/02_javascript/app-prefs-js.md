## Overview

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

The Application Preferences API is accessible through the [`glue.prefs`](../../../reference/glue/latest/application%20preferences/index.html) object.

## Get

To retrieve the stored application preferences for the current application, use the [`get()`](../../../reference/glue/latest/application%20preferences/index.html#API-get) method:

```javascript
const prefs = await glue.prefs.get();
```

This method resolves with an [`AppPreferences`](../../../reference/glue/latest/application%20preferences/index.html#AppPreferences) object with the following properties:

| Property | Type | Description |
|----------|------|-------------|
| `app` | `string` | The name of the application with which are associated the stored preferences. |
| `data` | `object` | The stored application preferences. |
| `lastUpdate` | `Date` | Timestamp of the last update of the application preferences. |

To retrieve the stored application preferences for a specific application, pass an application name to the [`get()`](../../../reference/glue/latest/application%20preferences/index.html#API-get) method:

```javascript
const appName = "clientlist";
const prefs = await glue.prefs.get(appName));
```

To retrieve the stored application preferences for all applications of the current user, use the [`getAll()`](../../../reference/glue/latest/application%20preferences/index.html#API-get) method:

```javascript
// Resolves with a collection of `AppPreferences` objects.
const prefs = await glue.prefs.getAll();
```

## Set

To set the preferences for the current application, use the [`set()`](../../../reference/glue/latest/application%20preferences/index.html#API-set) method:

```javascript
const prefs = { fontSize: 18 };

await glue.prefs.set(prefs);
```

The `set()` method replaces entirely the stored application preferences. All existing properties of the `AppPreferences` object associated with the app will be removed and replaced with the ones of the argument provided to the `set()` method.

To set the preferences for a specific application, pass an options object containing the app name as a second argument to the [`set()`](../../../reference/glue/latest/application%20preferences/index.html#API-set) method:

```javascript
const prefs = { fontSize: 18 };
const options = { app: "clientlist" };

await glue.prefs.set(prefs, options);
```

## Update

To update the preferences for the current application, use the [`udpate()`](../../../reference/glue/latest/application%20preferences/index.html#API-update) method:

```javascript
const prefs = { fontSize: 18 };

await glue.prefs.update(prefs);
```

The `update()` method modifies only the properties provided in the update object. All other existing properties of the `AppPreferences` object associated with the app will remain intact.

To update the preferences for a specific application, pass an options object containing the app name as a second argument to the [`update()`](../../../reference/glue/latest/application%20preferences/index.html#API-update) method:

```javascript
const prefs = { fontSize: 18 };
const options = { app: "clientlist" };

await glue.prefs.update(prefs, options);
```

## Clear

To remove the stored application preferences for the current application, use the [`clear()`](../../../reference/glue/latest/application%20preferences/index.html#API-clear) method:

```javascript
await glue.prefs.clear();
```

To remove the stored application preferences for a specific application, pass an application name to the [`clear()`](../../../reference/glue/latest/application%20preferences/index.html#API-clear) method:

```javascript
const appName = "clientlist";

await glue.prefs.clear(appName);
```

To remove the stored application preferences for all applications of the current user, use the [`clearAll()`](../../../reference/glue/latest/application%20preferences/index.html#API-clearAll) method:

```javascript
await glue.prefs.clearAll();
```

## Reference

For a complete list of the available Application Preferences API methods and properties, see the [Application Preferences API Reference Documentation](../../../reference/glue/latest/application%20preferences/index.html).