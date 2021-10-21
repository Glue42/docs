## Restart and Shutdown

The Application Management API is accessible through the [`glue.appManager`](../../../reference/glue/latest/appmanager/index.html) object.

*See the JavaScript [Application Management example](https://github.com/Glue42/js-examples/tree/master/app-management-example) on GitHub.*

To restart [**Glue42 Enterprise**](https://glue42.com/enterprise/), use the [`restart()`](../../../reference/glue/latest/appmanager/index.html#API-restart) method. This will close all running applications and their instances and then restart [**Glue42 Enterprise**](https://glue42.com/enterprise/):

```javascript
await glue.appManager.restart();
```

To shut down [**Glue42 Enterprise**](https://glue42.com/enterprise/), use the [`exit()`](../../../reference/glue/latest/appmanager/index.html#API-exit) method. This will close all running applications and their instances:

```javascript
await glue.appManager.exit();
```

## Managing Application Definitions at Runtime

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.12">

Application definitions can be imported, exported and removed at runtime using the [`InMemoryStore`](../../../reference/glue/latest/appmanager/index.html#InMemoryStore) object of the Application Management API. 

*Note that all application [`Definition`](../../../reference/glue/latest/appmanager/index.html#Definition) objects provided at runtime are stored in-memory and the methods of the `InMemoryStore` object operate only on them - i.e., the application definitions provided to [**Glue42 Enterprise**](https://glue42.com/enterprise/) through local or remote configuration files aren't affected.*

### Import

To import a list of application definitions at runtime, use the [`import()`](../../../reference/glue/latest/appmanager/index.html#InMemoryStore-import) method:

```javascript
const definitions = {
    {
        name: "my-app",
        type: "window",
        title: "My App",
        details: {
            url: "https://my-domain.com/my-app"
        }
    },
    {
        name: "my-other-app",
        type: "window",
        title: "My Other App",
        details: {
            url: "https://my-domain.com/my-other-app"
        }
    }
};
const mode = "merge";
const importResult = await glue.appManager.inMemory.import(definitions, mode);
```

The `import()` method accepts a list of [`Definition`](../../../reference/glue/latest/appmanager/index.html#Definition) objects as a first parameter and an import mode as a second. There are two import modes - `"replace"` (default) and `"merge"`. Using `"replace"` will replace all existing in-memory definitions with the provided ones, while using `"merge"` will merge the existing ones with the provided ones, replacing the application definitions with the same name. Use the `imported` property of the returned [`ImportResult`](../../../reference/glue/latest/appmanager/index.html#ImportResult) object to see a list of the successfully imported definitions and its `errors` property to see a list of the errors:

```javascript
const importedApps = importResult.imported;
const errors = importResult.errors;

importedApps.forEach(console.log);
errors.forEach(e => console.log(`App: ${e.app}, Error: ${e.error}`));
```

### Export

To export a list of already imported in-memory application definitions, use the [`export()`](../../../reference/glue/latest/appmanager/index.html#InMemoryStore-export) method:

```javascript
const definitions = await glue.appManager.inMemory.export();
```

### Remove

To remove a specific in-memory application definition, use the [`remove()`](../../../reference/glue/latest/appmanager/index.html#InMemoryStore-remove) method and provide the application name:

```javascript
await glue.appManager.inMemory.remove("my-app");
```

### Clear

To clear all imported in-memory definitions, use the [`clear()`](../../../reference/glue/latest/appmanager/index.html#InMemoryStore-clear) method:

```javascript
await glue.appManager.inMemory.clear();
```

## Listing Applications

To see a list of all applications available to the current user, use the [`applications()`](../../../reference/glue/latest/appmanager/index.html#API-applications) method:

```javascript
const applications = glue.appManager.applications();
```

### Specific Application

To get a reference to a specific application, use the [`application()`](../../../reference/glue/latest/appmanager/index.html#API-application) method and pass the name of the application as an argument:

```javascript
const app = glue.appManager.application("ClientList");
```

### Current Application Instance

To get a reference to the instance of the current application, use the [`myInstance`](../../../reference/glue/latest/appmanager/index.html#API-myInstance) property:

```javascript
const myInstance = glue.appManager.myInstance;
```

## Starting Applications

To start an application, use the [`start()`](../../../reference/glue/latest/appmanager/index.html#Application-start) method of the application object:

```javascript
const app = glue.appManager.application("ClientList");

const appInstance = await app.start();
```

The `start()` method accepts two optional parameters - a context object (object in which you can pass custom data to your app) and an [`ApplicationStartOptions`](../../../reference/glue/latest/appmanager/index.html#ApplicationStartOptions) object:

```javascript
const app = glue.appManager.application("ClientList");
const context = { selectedUser: 2 };
const startOptions = { hidden: true };

const appInstance = await app.start(context, startOptions);
```

## Listing Running Instances

To list all running instances of all applications, use the [`instances()`](../../../reference/glue/latest/appmanager/index.html#API-instances) method:

```javascript
// Returns a collection of the running instances of all apps.
glue.appManager.instances();
```

## Stopping Instances

To stop a running instance, use the [`stop()`](../../../reference/glue/latest/appmanager/index.html#Instance-stop) method of an instance object:

```javascript
await appInstance.stop();
```

## Events

### Shutdown Event

<glue42 name="addClass" class="colorSection" element="p" text="Available since Glue42 Enterprise 3.11">

The shutdown event provided by the Application Management API allows you to execute custom code before [**Glue42 Enterprise**](https://glue42.com/enterprise/) shuts down. The available time for the execution of your code is 60 seconds. The callback you provide for handling the event will receive an object with only one property (`restarting`) as an argument. Use it to determine whether [**Glue42 Enterprise**](https://glue42.com/enterprise/) is restarting or is shutting down.

```javascript
// The async code in the handler will be awaited up to 60 seconds
// before Glue42 Enterprise shuts down.
const handler = async (args) => {
    const isRestarting = args.restarting;
    
    if (isRestarting) {
        console.log("Restarting...");
    } else {
        await handleShutdown();
    };
};

glue.appManager.onShuttingDown(handler);
```

### Application Events

The set of applications defined for the current user can be changed at runtime. To track the events which fire when an application has been added, removed or updated, use the respective methods exposed by the Application Management API.

Application added event:

```javascript
const handler = application => console.log(application.name);

// Notifies you when an application has been added.
const unsubscribe = glue.appManager.onAppAdded(handler);
```

Application removed event:

```javascript
const handler = application => console.log(application.name);

// Notifies you when an application has been removed.
const unsubscribe = glue.appManager.onAppRemoved(handler);
```

Application updated event:

```javascript
const handler = application => console.log(application.name);

// Notifies you when an application configuration has been updated.
const unsubscribe = glue.appManager.onAppChanged(handler);
```

### Instance Events

To monitor instance related events globally (for all instances of all applications running in [**Glue42 Enterprise**](https://glue42.com/enterprise/)) or on an application level (only instances of a specific application), use the respective methods exposed by the Application Management API.

#### Global

The [`appManager`](../../../reference/glue/latest/appmanager/index.html#API) object offers methods which you can use to monitor instance events for all applications running in [**Glue42 Enterprise**](https://glue42.com/enterprise/). Get notified when an application instance has started, stopped, has been updated or when starting an application instance has failed. The methods for handling instance events receive a callback as a parameter which in turn receives the application instance as an argument. All methods return an unsubscribe function - use it to stop receiving notifications about instance events.

Instance started event:

```javascript
const handler = instance => console.log(instance.id);

const unsubscribe = glue.appManager.onInstanceStarted(handler);
```

Instance stopped event:

```javascript
const handler = instance => console.log(instance.id);

const unsubscribe = glue.appManager.onInstanceStopped(handler);
```

Instance updated event:

```javascript
const handler = instance => console.log(instance.id);

const unsubscribe = glue.appManager.onInstanceUpdated(handler);
```

#### Application Level

To monitor instance events on an application level, use the methods offered by the [`Application`](../../../reference/glue/latest/appmanager/index.html#Application) object. The methods for handling instance events receive a callback as a parameter which in turn receives the application instance as an argument.

Instance started event:

```javascript
const app = glue.appManager.application("ClientList");
const handler = instance => console.log(instance.id);

app.onInstanceStarted(handler);
```

Instance stopped event:

```javascript
const app = glue.appManager.application("ClientList");
const handler = instance => console.log(instance.id);

app.onInstanceStopped(handler);
```

## Reference

For a complete list of the available Application Management API methods and properties, see the [Application Management API Reference Documentation](../../../reference/glue/latest/appmanager/index.html).