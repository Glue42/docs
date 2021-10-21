# How to Glue42 Enable Your App

## Referencing

To use any **Glue42 Enterprise** functionality, you need to reference either the Glue42 Angular library as an `npm` package, or directly the Glue42 JavaScript library as a standalone file or as an `npm` package.

*For more info on referencing the Glue42 JavaScript library, see the [JavaScript](../javascript/index.html) section on referencing and initializing Glue42 in your apps.*

### As an NPM Package

The Glue42 Angular library is available as an `npm` package called `@glue42/ng-glue`. To install it, navigate to the root directory of your project and run:

```cmd
npm install @glue42/desktop @glue42/ng-glue
```

### As a Standalone File

You can also reference directly the Glue42 JavaScript library as a standalone file. If you are using a Glue42 installer, you can find the `.js` files in the `%LOCALAPPDATA%\Tick42\GlueSDK\GlueJS\js\web` folder and copy them in your project folder. 

```javascript
import Glue, { Glue42 } from "../../../lib/tick42-glue"; 
```

## Initialization

Once installed, you need to import the Glue42 Angular module and invoke the static method `withConfig()` passing an optional configuration object to it. This method initializes an instance of Glue42. Your Glue42 will be initialized based both on the properties of the configuration object and on the [default values](../../../../reference/glue/latest/glue/index.html#Config) of the respective configuration properties. Depending on whether a Glue42 API will be initialized or not, `NgGlue42Module` will include/exclude the appropriate service as a provider.

Example initialization, where `AppManagerService` will be available as a provider and `LayoutsService` will not:

```js
import { NgGlue42Module } from "@glue42/ng-glue";
 
@NgModule({
    imports: [
        NgGlue42Module.withConfig({ appManager: "full", layouts: false })
    ]
})
export class AppModule { }
```

This is an example of how you can create a service which when invoked will initialize Glue42:

```javascript
import Glue, { Glue42 } from '../../../lib/tick42-glue';

@Injectable()
export class GlueService {

    private _glue: Glue42.Glue;

    public async initialize(): Promise<void> {
        this._glue = await Glue();
        (window as any).glue = this._glue;
    }

}
```

## Using the Library

You can start using the Glue42 Angular library by injecting any of the provided APIs into your components and services.

**Example**

- In one app, listen for a certain method to be registered and immediately invoke it when it becomes available:

```js
import { Component, OnInit } from "@angular/core";
import { InteropService } from "@glue42/ng-glue";
import { mergeMap } from "rxjs/operators";
 
@Component({
    selector: "app-root",
    templateUrl: "app.component.html"
});

export class AppComponent implements OnInit {
 
    constructor(public interopService: InteropService) { };
 
    ngOnInit(): void {
        this.interopService.methodRegistered({ methodName: "Sum" })
            .pipe(
                mergeMap(
                    () => this.interopService.invoke("Sum", { a: 37, b: 5 });
                )
            ).subscribe({
                next: result => console.log(`The result of 37 + 5 is ${result.returned.answer}`)
            });
    
    }
}
```

- And register the method in another app:

```js
this.interopService.register("Sum", ({ a, b }) => ({ answer: a + b }))
    .subscribe({
        next: method => console.log(`Method "${method.name}" is registered.`),
        error: error => console.log(`Couldn't register method "Sum"`, error)
    });
```

## Application Configuration

To show your app in the **Glue42 Enterprise** App Manager dropdown menu, you need to create an application configuration `.json` file. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` should be replaced by the environment and region of your **Glue42 Enterprise** copy (e.g., `T42-DEMO`).

Below is an example configuration:

```json
[
    {
        "details": {
            "mode": "tab",
            "name": "angular_example",
            "url": "http://localhost:4200"
        },
        "ignoreSaveOnClose": true,
        "name": "Angular_Example",
        "title": "Angular Example",
        "caption": "Angular Example",
        "type": "window"
    }
]
```

*See also our Angular example in [GitHub](https://github.com/Tick42/angular-example), demonstrating Glue42 initiation and integration in a standard Angular app.*

# Glue42 Concepts

# Application Management

## Accessing the Service

The **Application Management** service can be accessed by injecting `AppManagerService` in any Angular component or service.

*See the Angular [Application Management example](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/app-manager) on GitHub.*

```typescript
import { AppManagerService } from "@glue42/ng-glue";

@Component({
    selector: "app-root"
    templateUrl: "app.component.html"
})
export class AppComponent {
    constructor(appManagerService: AppManagerService) { }
}
```

*If you still need to use the JavaScript [`Application Management API`](../javascript/index.html) directly, you can access it via `appManagerService.api`.*

## Discovering Applications

You can use `appManagerService.applications()` to construct an `Observable<Application[]>`. When you subscribe to it, it will emit all available applications in **Glue42 Enterprise**. The `Observable` emits the new state of the applications collection as an array every time an application is added, changed or removed.

Method signature:

```typescript
applications(): Observable<Application[]>
```

Example:

```typescript
appManagerService.applications()
    .pipe(
        map((applications: Application[]) => applications.filter(x => x.hidden === false))
    )
    .subscribe({
        next: (applications: Application[]) => {
            // use the applications
        }
    });
```

If you want to observe existing or newly added applications and how they change, call `appManagerService.applicationAddedOrChanged()`. This method accepts an optional application name as an argument if you want to match a certain application. If a filter is provided and an existing application matches it, it will be emitted immediately. The returned `Observable<Application>` will also emit every time the application is changed. If no filter is provided, the `Observable` will just emit every new (or edited) application that has been added to **Glue42 Enterprise**.

Method signature:
```typescript
applicationAddedOrChanged(name?: string): Observable<Application>
```
Example:

```typescript
appManagerService.applicationAddedOrChanged("clientlist")
    .subscribe({
        next: application => appManagerService.start(application.name).subscribe()
    });
```

Similarly, by calling `appManagerService.applicationRemoved()` you can observe whether an application has been removed and is no longer available. This method can also accept an optional application name and returns an `Observable<Application>` which will emit every `Application` that is no longer available in **Glue42 Enterprise**.

Method signature:

```typescript
applicationRemoved(name?: string): Observable<Application>
```

Example:

```typescript
appManagerService.applicationRemoved("clientlist")
    .subscribe({
        next: application => {
            // do something with application
        }   
    });
```

## Starting Applications

To start an application, construct an `Observable<AppInstance>` by calling `appManagerService.start()` and providing an application name, an optional initial context and application options. When you subscribe to it, the `Observable` will attempt to start the application and upon success, will emit once(the new AppInstance) and then complete.

Method signature:

```typescript
start(name: string, applicationContext?: any, options?: ApplicationStartOptions): Observable<AppInstance>
```
Example:

```typescript
appManagerService.start("clientlist")
    .subscribe({
        next: appInstance => {
            // use instance
        }
    });
```

## Discovering Instances

You can use `appManagerService.appInstances()` to construct an `Observable<AppInstance[]>`. When you subscribe to it, it will emit all running application instances in **Glue42 Enterprise**. The `Observable` emits the new state of the application instances collection as an array every time an instance is started or stopped. You can pass an application name as an optional filter.

Method signature:

```typescript
appInstances(applicationName?: string): Observable<AppInstance[]>
```
Example:

```typescript
appManagerService.appInstances("clientlist")
    .subscribe({
        next: instances => {
            // use instances
        }
    });
```

If you want to observe the existing or newly started instances of applications, call `appManagerService.appInstanceStarted()`. When you subscribe to the returned `Observable<AppInstance>`, it will start emitting when a new instance of an application is started or when an already started instance is updated. This method accepts an optional application name if you want to match only the instances of a certain application. In this case, when you subscribe to the returned `Observable`, all currently running instances matching the filter will be emitted consecutively. 

Method signature:

```typescript
appInstanceStarted(applicationName?: string): Observable<AppInstance>
```

Similarly, you can listen for instances that stop running.

Method signature:

```typescript
appInstanceStopped(applicationName?: string): Observable<AppInstance>
```

Example:

```typescript
appManagerService.appInstanceStopped("clientlist")
    .subscribe({
        next: appInstance => {
            // use instance
        }
    });
```

# Shared Contexts

## Accessing the Service

The **Shared Contexts** service can be accessed by injecting `SharedContextService` in any Angular component or service.

```typescript
import { SharedContextsService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(sharedContextsService: SharedContextsService) { }
}
```

*If you still need to use the JavaScript [`Shared Contexts API`](../javascript/index.html) directly, you can access it via `sharedContextsService.api`.*

## Creating a Context

To initially create a context, construct an `Observable<Context>` by calling `sharedContextService.set()` or `sharedContextService.update()` with the context name and properties. If the context doesn't exist yet, it makes no difference which method you will call. When you subscribe to the `Observable<Context>`, it will set the new context, emit it once and then complete.

```typescript
sharedContextsService.set(
    "app-styling",
    {
        backgroundColor: "red",
        alternativeColor: "green"
    })
    .subscribe({
        next: context => console.log(context.data)
    });
    // { backgroundColor: "red", alternativeColor: "green" }
```

## Updating a Context

To update an existing context, construct an `Observable<Context>` by calling `sharedContextService.update()` with the context name and the properties you need to update. When you subscribe to it, the `Observable<Context>` will update the context - new properties (context keys) will be added to the context object, existing ones will be updated, and you can also remove context keys by setting them to `null`. Then the context will be emitted in its newest state.

```typescript
sharedContextsService.update(
    "app-styling",
    {
        backgroundColor: "blue",
        alternativeColor: null
    })
    .subscribe({
        next: context => console.log(context.data)
    });
    // { backgroundColor: "blue" }
```

## Replacing a Context

Replacing a context is nothing more than re-creating it. So, if you need to change a context completely, rather than just update it, reset it by calling `sharedContextService.set()` with the context name and properties. This will construct an `Observable<Context>`. When you subscribe to it, it will remove all previous context properties and replace them with the new ones you provide. Then, the newest context will be emitted.

```typescript
sharedContextsService.set(
    "app-styling",
    {
        generalTheme: "dark",
        BackgroundOpacity: 50
    })
    .subscribe({
        next: context => console.log(context.data)
    });
    // { generalTheme: "dark", BackgroundOpacity: 50 }
```

## Subscribing for Context Updates

To subscribe for context updates, call `sharedContextsService.context()` and provide a context name. This method returns an `Observable<Context>`, which emits every time the context changes. The emitted object contains the newest state of the context, the delta (what has changed) and the properties removed from it.

```typescript
sharedContextsService.context("app-styling")
  .subscribe({
      next: context => console.log(context.data)
  });

sharedContextsService.set("app-styling", { theme: "dark" })
    .subscribe();
// { theme: "dark" }

sharedContextsService.update("app-styling", { theme: "light" })
    .subscribe();
// { theme: "light" }
```

## Unsubscribing

To stop listening for context changes for a certain context, you need to call the `unsubscribe()` function of the respective `Subscription`.

```typescript
const styleContextSubscription = sharedContextsService.context("app-styling")
    .subscribe({
        next: context => console.log(context.data)
    });

sharedContextsService.set("app-styling", { theme: "dark" })
    .subscribe();
// { theme: "dark" }

styleContextSubscription.unsubscribe();

sharedContextsService.update("app-styling", { theme: "light" })
    .subscribe();
// no logging - the subscription is no longer active.
```

# Channels

## Adding Channels to Your Application

To add the channel selector to your window, you need to set `allowChannels` to `true` in your application configuration file:

```json
{
    "title": "Client List 🔗",
    "type": "window",
    "name": "channelsclientlist",
    "icon": "https://dev-enterprise-demos.tick42.com/resources/icons/clients.ico",
    "details": {
        "url": "https://dev-enterprise-demos.tick42.com/client-list-portfolio-contact/#/clientlist",
        "mode": "tab",
        "allowChannels": true
    }
}
```

## Defining Channels

You can define any number of channels in **Glue42 Enterprise** for your applications to use - all you need to do is to configure them in the `channels.json` file in the `%LocalAppData%\Tick42\GlueDesktop\config` folder. Here is an example of adding a custom purple channel to an already existing list of channels in **Glue42 Enterprise**:

```json
{
    "name": "Dark Purple",
    "meta": {
        "color": "#6400b0"
    }
}
```

![Custom Channel](../../../../images/channels/custom-channel.gif)

## Accessing the Service

The **Channels** service can be accessed by injecting `ChannelsService` in any Angular component or service.

```typescript
import { ChannelsService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(channelsService: ChannelsService) { }
}
```

*If you still need to use the JavaScript [`Channels API`](../javascript/index.html) directly, you can access it via `channelsService.api`.*

## Subscribing for Data

To track the context of the current channel, subscribe to `channelsService.channelContext`.

```typescript
channelsService.channelContext
  .subscribe({
    next: context => console.log(context.data)
  });
  // { backgroundColor: "red", alternativeColor: "green" }
```
Subscribers will be notified each time when:

- the `data` of the channel you are currently on is updated;
- the user has switched the channel and you are receiving a snapshot of the new channel `data`; 
- your app is now not joined to any channel (e.g., the user has deselected the current channel). In this case, both `data` and `channelInfo` will be `undefined`;

## Publishing Data

To update the context data of the current channel, construct an `Observable<void>` by calling `channelsService.publish()`. When you subscribe to it, the context of the channel the application is currently on will be updated accordingly. This method accepts new data as an argument.

```typescript
channelsService.publish({color: "green"})
  .subscribe({
      next: () => console.log("The current context was updated."),
      error: (error) => console.error("The window is not part of any channel!", error)
  });
```

## Current Channel

To get the current channel name, or to track how your applications moves between different channels, subscribe to `channelsService.currentChannel`. Subscribers will be notified each time the channel is changed.

```typescript
channelsService.currentChannel
  .subscribe({
    next: channelName => console.log(channelName)
  });
    // Green
```

# Interop

## Accessing the Service

The **Interop** service can be accessed by injecting `InteropService` in any Angular component or service.

*See the Angular [Interop example](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/interop) on GitHub.*

```typescript
import { InteropService } from "@glue42/ng-glue";
 
@Component({
    selector: "app-root",
    templateUrl: "app.component.html"
});
export class AppComponent {
    constructor(interopService: InteropService) { }
}
```

*If you still need to use the JavaScript [`Interop API`](../javascript/index.html) directly, you can access it via `interopService.api`.*

## Method Registration

To offer a method to other applications, call either `interopService.register()` or `interopService.registerAsync()` to construct an `Observable<MethodDefinition>`. When you subscribe to it, it will register a new synchronous/asynchronous Interop method. Pass the definition of the method and a callback to handle invocations from clients. Subscribers can retrieve the registered method, or handle an error if it occurs.

Methods signature:

```typescript
register(methodDefinition: MethodDefinitionParam, handler: RegisterMethodHandler): Observable<MethodDefinition>
```

```typescript
registerAsync(methodDefinition: MethodDefinitionParam, handler: RegisterMethodAsyncHandler): Observable<MethodDefinition>
```

Example:

```typescript
interopService.register("Sum", ({ a, b }) => ({ answer: a + b }))
    .subscribe({
        next: method => console.log(`Method "${method.name}" is registered.`),
        error: error => console.error("Couldn't register method 'Sum'", error)
    });
```

## Method Invocation

To invoke a method offered by another application, call `interopService.invoke()` and pass the name of the method, its arguments as an object and optionally provide a target and invocation options. Subscribe to the returned `Observable<InvocationResult<T>>` to invoke the Interop method and retrieve the result of the invocation, or handle an error if it occurs.

Method signature:

```typescript
invoke<T>(
    methodDefinition: MethodDefinitionParam,
    argument?: { [key: string]: any },
    settings?: InvocationSettings
  ): Observable<InvocationResult<T>>;
```

Example:

```typescript
interopService.invoke("Sum", { a: 37, b: 5 })
    .subscribe({
        next: ({ returned }) => console.log(`The result of 37 + 5 is ${returned.answer}`),
        error: console.error
    });
```

## Discovery

**Discovering Methods and Servers**

The **Interop** service provides methods which help to discover Interop methods and servers in a completely reactive way.

| Function          | Description |
|-------------------|-------------|
|`methods(filterOptions?: MethodFilter): Observable<ServerMethod[]>`|When subscribed to, the returned `Observable<ServerMethod[]>` will emit all registered server methods as an array and will emit every time a method is registered or unregistered. You can pass an optional method filter to target certain methods.|
|`methodRegistered(filterOptions?: MethodFilter): Observable<ServerMethod>`|Emits every time a method is offered by a server. You can pass an optional method filter to target certain methods. If you pass a filter and set all its options (`methodName` *and* `applicationName`) and it matches an already existing server method, this method will be emitted immediately.|
| `methodUnregistered(filterOptions?: MethodFilter): Observable<ServerMethod>`|Emits when an existing method is removed by a server. |
| `servers(): Observable<Server[]>`| When subscribed to, the returned `Observable<Server[]>` will emit all available Interop servers as an array and will emit every time a server is added or removed. |
| `serverAdded(applicationName?: string): Observable<Server>`|Emits when an application offering methods is discovered. You can provide an application name as an optional filter. If a filter is provided and an existing server matches it, it will be emitted immediately.|
|`serverRemoved(applicationName?: string): Observable<Server>`| Emits when an application no longer offers Interop methods. |


Examples:

A subscriber can retrieve all methods that are registered by the application "Client List":

```typescript    
interopService.methods({
        applicationName: "clientlist"
    }).subscribe({
        next: (methods: ServerMethod[]) => {
            // use server methods
        }
    });
```

A subscriber can retrieve all Interop servers:

```typescript
interopService.servers()
    .subscribe({
        next: (servers: Server[]) => {
            // use servers
        }
    });
```

A subscriber can retrieve a newly registered method by the application "Client List":

```typescript
interopService.methodRegistered({ applicationName: "clientlist" })
    .pipe(
        map(({ method }) => method.name)
    )
    .subscribe({
        next: (methodName: string) => {
            console.log(`Application "Client List" registered a method ${methodName}`);
        }
    });
```

Or a subscriber can be notified when a specific method is no longer offered by an application.

```typescript
interopService.methodUnregistered({ methodName: "Sum" })
    .subscribe({
        next: ({ method, server }: ServerMethod) => {
            console.log(`Application ${server.applicationName} stopped offering method ${method.name}`);
        }
    });
```

## Streaming

### Subscribing to Streams

To subscribe for stream data using the **Interop** service, call `interopService.readStream()`.

Method signature:

```typescript
readStream(methodDefinition: MethodDefinitionParam, parameters?: SubscriptionParams): Observable<StreamData>
```

The constructed `Observable<StreamData>` will emit data from the publisher, until the publisher closes the stream, closes your subscription or an error occurs.

```typescript
const lastTradesSubscription = interopService.readStream(
    "MarketData.LastTrades", 
    { 
        arguments: { symbol: "GOOG" }, 
        target: "all" 
    })
    .subscribe({
        next: ({ data }) => console.log(data.lastTradePrice),
        error: console.error,
        complete: () => console.log('MarketData.LastTrades subscription closed')
    });
```

To stop listening for stream data, unsubscribe from `Observable<StreamData>`.

```typescript
lastTradesSubscription.unsubscribe();
```

### Publishing Streams

To start publishing data, call `interopService.createStream()` to construct an `Observable<Stream>` and subscribe to it. It will create the defined Interop stream. Subscribers can retrieve the stream, or handle an error if it occurs.

Method signature:

```typescript
createStream(methodDefinition: MethodDefinitionParam, options?: StreamOptions): Observable<Stream>
```

And here is an example of creating an Interop stream:

```typescript
interopService.createStream({
    name: "MarketData.LastTrades",
    displayName: "Publishes last trades for a symbol",
    objectTypes: ["Symbol"],
    accepts: "String symbol",
    returns: "String symbol, Double lastTradePrice, Int lastTradeSize"
})
.subscribe({
    next: (stream) => {
        setInterval(() =>
            stream.push(
                {
                    symbol: "GOOG",
                    lastTradePrice: 700.91,
                    lastTradeSize: 10500
                }),
            5000)
    },
    error: console.error
});
```

# Activities

## Activity Participants

Each activity instance has a **single owner window** and can optionally have one or more **helper windows**. The owner window controls the lifetime of the activity - if the owner is closed, the activity and all other windows are closed as well. The activity is running as long as the owner is running. Helper windows support the owner of the activity and may be defined in the activity type configuration or joined to the activity at runtime.

![Activity](../../../../images/activities/activities.png)

## Configuring an Activity

Activity applications are defined in the same manner as regular applications. You can configure an activity by creating an activity configuration `.json` file and place it in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` should be replaced by the environment and region of your **Glue42 Enterprise** copy (e.g., `T42-DEMO`).

```json
[
    {
        "title": "Clients",
        "type": "activity",
        "name": "clients",
        "icon": "http://localhost:22080/resources/icons/client-list.ico",
        "hidden": false,
        "details": {
            "initialContext": {
                "context": "someContext"
            },
            "layout": {
                "mode": "pixels"
            },
            "owner": {
                "type": "clientlist",
                "name": "ClientList",
                "left": 20,
                "top": 20,
                "width": 400,
                "height": 400
            },
            "windows": [    
                {
                    "type": "clientportfolio",
                    "name": "ClientPortfolio",
                    "left": 20,
                    "top": 20,
                    "width": 400,
                    "height": 400
                }
            ]
        }
    },
    {
        "title": "Client List",
        "type": "window",
        "name": "ClientList",
        "hidden": true,
        "details": {
            "url": "http://localhost:22080/client-list-portfolio-contact/dist/#/clientlist",
            "mode": "html"
        },
        "activityTarget": {
            "enabled": true,
            "windowType": "clientlist"  
        }                               
    },
    {
        "title": "Client Portfolio",
        "type": "window",
        "name": "ClientPortfolio",
        "hidden": true,
        "details": {
            "url": "http://localhost:22080/client-list-portfolio-contact/dist/#/clientportfolio/",
            "mode": "html"
        },
        "activityTarget": {
            "enabled": true,
            "windowType": "clientportfolio"
        }
    }
]
```

- `owner` - the owner window of the activity;
- `windows` - array of helper windows;
- `windowType` - unique identifier within the **Activities** API (defaults to the application name);

Only the `owner` property is required. It is useful to specify the `activityTarget` property in order to describe how your application will look when it is an activity window. The same is valid for the `layout` property, which specifies whether the window dimensions are in `pixels` (default) or relative to the screen (`percents`). For more details, you can see the application configuration [schema](../../../../assets/configuration/application.json).

## Accessing the Service

The **My Activity** service can be accessed by injecting `MyActivityService` in any Angular component or service.

```typescript
import { MyActivityService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(myActivityService: MyActivityService) { }
}
```

*If you still need to use the JavaScript [`Activities API`](../javascript/index.html) directly, you can access it via `myActivityService.api`.*

## Detecting Activities

A window is not necessarily aware whether it was instantiated as part of an activity or as a standalone window, so the best practice is to design windows to be able to either pick context from the activity, or by registering [Interop methods](../../interop/javascript/index.html#method_registration).

If your window is part of an activity, `myActivityService.api.window` will hold a valid activity window reference on which you can expect the `isOwner` property (if `false`, the window is a helper window).

To check whether a window joins or leaves an activity use `myActivityService.joinedActivity()` or `myActivityService.leftActivity()` methods.

Methods Signature:

```typescript
joinedActivity(): Observable<Activity>
```

```typescript
leftActivity(): Observable<Activity>
```

Example:
```typescript
myActivityService.leftActivity()
    .subscribe({
        next: (activity) => {
            console.log("Window left an activity ", activity);
        }
    });
```

## Managing Activity Context

### Subscribing for Context Updates

To track the context updates of the current activity, call `myActivityService.context()` and subscribe to the returned `Observable<ActivityContext>`.

Method Signature:

```typescript
context(): Observable<ActivityContext>
```

Example:

```typescript
myActivityService.context()
    .subscribe({
        next: ({ context, delta, removed, activity }) => {
            // react to context updates
        }
    });
```

- `context` is the current context object of the activity;
- `delta` is an object containing only the changes between the previous and the latest contexts;
- `removed` is an array containing the names of the properties which were removed from the context;
- `activity` is the activity object;

### Updating a Context

To update the context of the current activity, call `myActivityService.updateContext()` which constructs an `Observable<ActivityContext>`. When you subscribe to it, it will update the context. Subscribers can retrieve the changed context.

Method Signature:

```typescript
updateContext(context: any): Observable<ActivityContext>
```

For example, if the current context is:

```typescript
const context = {
    // client information
    party: {
        pId: "12345",
        gId: "56789"
    },
    // instrument information
    instrument: {
        ric: "VOD.L",
        bpod: "VOD.LN",
        bbgTerminal: "VOD LN Equity"
    }
};
```

And we select another `party` and remove the selected `instrument`:

```typescript
myActivityService.updateContext({
    party: { pId: "34567", gId: "01234" },
    instrument: null
}).subscribe({
    next: ({ context, delta, removed }) => {
        console.log("Context: ", context);
        console.log("Delta: ", delta);
        console.log("Removed: ", removed);
    }
});
```

The subscriber to `updateContext()`, as well as any subscriber to `context()`, will receive the following arguments:

```typescript
// new context
{ 
    party: {
        pId: "34567", // note the change
        gId: "01234"  // note this one as well
    }
};

//object, containing the changes between the new and the old context
{
    party: {
        pId: "34567",
        gId: "01234"
    },
    instrument: null
};

// the properties removed from the context
[ "instrument" ]
```

### Replacing a Context

To replace the context of the current activity, use `myActivityService.setContext()`.

Method Signature:

```typescript
setContext(context: any): Observable<ActivityContext>
```

## Managing Activity Windows

### Creating Activity Windows

Use either `myActivityService.createWindow()` or `myActivityService.createStackedWindows()`. Both methods construct an `Observable<ActivityWindow>`/`Observable<ActivityWindow[]>` that, when subscribed will create the configured activity windows.

Methods Signature:

```typescript
createWindow(windowType: string | WindowDefinition): Observable<ActivityWindow>
```

```typescript
createStackedWindows(windowTypes: (string | WindowDefinition)[], timeout?: number): Observable<ActivityWindow[]>
```

When creating a window, you can pass an initial context, window position and/or style to it, just like in the [Window Management](../../../windows/window-management/javascript/index.html#window_management/javascript/index.html) API `windows.open()` call:

```typescript
myActivityService.createWindow({
    isIndependent: false,
    name: "clientportfolio",
    type: "clientportfolio",
    relativeTo: "clientlist",
    relativeDirection: "bottom",
    height: 300
})
.subscribe({
    next: (window) => {
        window.activate(true); // you can use the Window Management API here
    }
});
```

![Created activity window](../../../../images/activities/activity-create.gif)

### Activity Layouts

Activities have special layouts (arrangement of windows/apps) that can be saved and later restored.

For more information on how to do that, see the [Activity Layouts](../../../windows/layouts/javascript/index.html#activity_layouts) section of the Layouts documentation.

# Window Management

## Accessing the Service

The **Windows Management** service can be accessed by injecting `WindowsService` in any Angular component or service.

*See the Angular [Glue42 Window](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/gd-window) and [Window Management](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/windows) examples on GitHub.*

```typescript
import { WindowsService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(windowsService: WindowsService) { }
}
```

*If you still need to use the JavaScript [`Windows Management API`](../javascript/index.html) directly, you can access it via `windowsService.api`.*

## Opening Windows

To open a new Glue42 browser window, construct an `Observable<GDWindow>` using the `windowsService.open()` method. When you subscribe to it, it will open the window and emit it.

Method signature:

```typescript
open(
    name: string,
    url: string,
    options?: WindowSettings): Observable<GDWindow>;
```

Styles are documented in the [Window Settings](#window_settings).

```typescript
windowsService.open(
    "glue-docs",                            // window name
    "https://docs.glue42.com/",             // URL
    {
        top: 150,                           // bounds
        left: 150,
        allowClose: false,
        allowCollapse: true
    })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
``` 

**Relative Positions**

When opening a new window, you can position it relatively to another window.
Here is an example:

```javascript
const clientWindow = windowsService.api.my();

// open a "portfolio" window relative to the "client" window
windowsService.open(
    "portfolio", 
    "www.portfolio.com", 
    {
        relativeTo: clientWindow.id,
        relativeDirection: "right"
    })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

And the result:

![Opening a window relative to another window](../../../../images/window-management/relative-to.gif)


## Window Modes

Glue42 supports 3 different window modes: flat, tab and HTML. The window mode is controlled by the [`mode`](../../../../reference/glue/latest/windows/index.html#WindowSettings-mode) window setting, which can be specified in the application configuration settings or be part of the open a window/start an app call:

Application configuration settings:

```typescript
{
    "title": "Client List",
    "type": "window",
    "name": "clientlist",
    "icon": "http://localhost:22080/resources/icons/client-list.ico",
    "details": {
        "url": "http://localhost:22080/index.html",
        "mode": "html"
    }
}
```

Open a window call:
```typescript
windowsService.open("portfolio", "www.portfolio.com", {
        mode: "tab"
    })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

**Flat window**

Flat windows have framework provided frame that includes tab header and buttons. 

![Image of a flat window](../../../../images/window-management/window-mode-flat.png)

**Tab window**

Tab windows have framework provided frame that includes tab header and buttons. They can be tabbed with other tab windows.

![Image of a tab window](../../../../images/window-management/window-mode-tab.png)

**HTML window**

HTML windows do not have a frame - the HTML application is responsible for providing adequate user experience.

![Image of an HTML window](../../../../images/window-management/window-mode-html.png)

## Window Settings

*The provided values will override the default values.*

| Name  | Type | Description  | Default  | Supported By | Runtime Update Support |
|---|---|---|---|---|---|
| `allowClose` | bool | If `false`, the window will not contain a close button   | `true`  |  All  | Flat and HTML |
| `allowTabClose` | bool | If `false`, the tab header will not contain a close button   | `true`  |  Tab  | None |
| `allowCollapse` | bool | If `false`, the window will not contain a collapse button    | `true`  |  All | Flat and HTML |
| `allowForward` | bool | If `false`, the window will not contain an activity related forward button |  `true` | HTML | None |
| `allowMaximize` | bool | If `false`, the window will not contain a maximize button | `true` | All  | Flat and HTML |
| `allowMinimize` | bool | If `false`, the window will not contain a minimize button | `true` | All  | Flat and HTML |
| `allowUnstick` | bool | If `false`, the window will not unstick from other windows  | `true`  | All  | None |
| `allowLockUnlock` | bool | If `false`, the window will not contain a lock/unlock button | `false` | All | Flat and HTML |
| `autoSnap` | bool | If `true`, when moving the window operation ends, the window will snap to one of the approaching edges of another window (if any of the approaching edges are marked with red)| `true` | All  | None |
| `autoAlign` | bool | If `true`, a snapped window will adjust its bounds to the same width/height of the window it has stuck to, and/or will occupy the space between other windows (if any)  | `true` | All  | None |
| `base64ImageSource` | string | Image as Base64 string that will be used as a taskbar icon for the window. The supported formats are `png`, `ico`, `jpg`, `apng`. | `-` | All | All |
| `borderColor` | string | Can be a color name such as "Red", or a hex-encoded RGB or ARGB value  | `-`  | Flat | None |
| `buttonsVisibility` | string | Controls the button visibility. Can be set to `off`, `onFocus`, `onDemand` or `always`. | `onDemand` | HTML | None |
| `collapseHeight` | number | Defines the height of the window when collapsed | `System titlebar height`  | Flat and HTML | Flat and HTML |
| `devToolsEnable`| bool | If `true`, allows opening a developer console (using F12) for the new window | `true` | All | None |
| `downloadSettings`| Object| Object that defines file download behavior in the window | `-` | All | None |
| `downloadSettings.autoSave`| bool | If `true`, will auto save the file (without asking the user where to save it). If `false`, a system save dialog will appear. | `true` | All | None |
| `downloadSettings.autoOpenPath`| bool| If `true`, will open the folder that contains the downloaded file after the download is completed | `false` | All | None |
| `downloadSettings.autoOpenDownload`| bool | If `true`, will open the download file after the download is completed | `false` | All | None |
| `downloadSettings.enable`| bool| If `true`, enables the window to download files | `true` | All | None |
| `downloadSettings.enableDownloadBar`| bool| If `true`, a download bar tracking the progress will appear on the bottom of the window when downloading. If `false`, the download process will be invisible | `true` | All | None |
| `downloadSettings.path`| string| Path where the downloaded file will be saved. Due to security reasons, it is only possible to provide two download paths: the Windows "Temp" or "Downloads" folder | `-` | All | None |
| `isCollapsed` | bool | If `true`, the window will start collapsed | `false` | All | None |
| `isSticky` | bool | If `true`, the window will stick to other Glue42 windows forming groups | `true` | All | None |
| `focus` | bool | If `false`, the window will not be on focus when created  | `true`  |  All | All |
| `hasMoveAreas` | bool | If `false`, the window cannot be moved  | `true` | Flat and HTML  | Flat |
| `hasSizeAreas` | bool | If `false`, the window cannot be resized by dragging its borders, maximizing, etc.  | `true` | Flat and HTML | Flat |
| `hidden` | bool |  If `true`, the window will be started as a hidden window | `false`  | All  | All |
| `historyNavigationEnabled` | bool | If `true`, this will allow the users to navigate back (CTRL+Left) and forward (CTRL+Right) through the web page history | `GLOBAL CONFIG` | All | None |
| `maxHeight` | number | Specifies the maximum window height  | `-` | All  | All |
| `maxWidth` | number | Specifies the maximum window width  | `-`  | All  | All |
| `minHeight` | number | Specifies the minimum window height  | `-` | All  | All |
| `minWidth` | number | Specifies the minimum window width | `-` | All | All |
| `mode` | string | Glue42 Window type. Possible values are `flat`, `tab` and `html`. | `flat`  | - | None |
| `moveAreaThickness` | string | How much of the window area is to be considered as a moving area (meaning you can move the window using it). The string value corresponds to the left, top, right and bottom borders.  |  `0, 12, 0, 0` | HTML | None |
| `moveAreaTopMargin` | string | The Glue42 Window may contain a move area thickness top margin. The margin is related to the top border of `moveAreaThickness` only. The string value corresponds to the left, top, right and bottom  | `0, 0, 0, 0`  | HTML | None |
| `onTop` | bool |  If `true`, the window will appear on top of the z-order  | `false` | All | None |
| `relativeTo` | string | The ID of the window that will be used to relatively position the new window. Can be combined with `relativeDirection`  | `-`  | All  | None |
| `relativeDirection` | string |  Direction (`bottom`, `top`, `left`, `right`) of positioning the window relatively to the `relativeTo` window. Considered only if `relativeTo` is supplied | `right`  | All | None |
| `showInTaskbar` | bool | If `false`, the window will not appear on the Windows taskbar  | `true`  | All | None |
| `showTitleBar` | bool | Determines whether the window will have a title bar  | `true`  | Flat | None |
| `sizeAreaThickness` | string | How much of the window area is to be considered as a sizing area (meaning you can resize the window using that area). The string value corresponds to the left, top, right and bottom borders  |  `5, 5, 5, 5`  | HTML  | None |
| `snappingEdges` | string | Specifies the active Glue42 window snapping edges. Possible values are: `top`, `left`, `right`, `bottom`, `all` or any combination of them (e.g., `left, right`)  | `all` | All  | None |
| `startLocation` | string | Specifies the start window location. Possible options are `Automatic` (The Glue42 Window decides where the window will be positioned) and `CenterScreen`  | `Automatic`  | All  | None |
| `stickyFrameColor` | string | Specifies the Glue42 window frame color. Accepts hex color as string (e.g. `#666666`) or named HTML colors (e.g. `red`) | `#666666`  | All  | All |
| `stickyGroup` | string | If set, the Glue42 window can only stick to windows that have the same group.  | `Any` | All | None |
| `tabGroupId` | string | Specifies the tab group ID. If two or more tab windows are defined with the same ID, they will be hosted in the same tab window  | `Automatic`  | Tab  | None |
| `tabIndex` | number | Specifies the tab position index. Tab windows in the same tab group are ordered by their position index. Use negative index to make the tab active.  | `-` | Tab  | None |
| `tabSelected` | bool | Tab is selected  | `false`  | Tab  | None |
| `tabTitle` | string | The tab title  | `""`  | Tab  | Tab |
| `tabToolTip` | string | The tab tooltip  | `""`  | Tab  | Tab |
| `title` | string | Sets the window title. To work properly, there should be a title HTML tag in the page | `-` | All | All |
| `url` | string | The URL of the app to be loaded in the new window | `-` | All | All |
| `useRandomFrameColor` | bool | If `true`, this will set a random (from a predefined list of colors) frame color to the new window | `false`| All | None |
| `windowState` | string | If set, the window will start in the specified state (maximized, minimized, normal) | `normal` | All | All |
| `windowName` | string | The name of the window. | `-` | All | None |

You can provide window settings per window by:

- using the application configuration settings:

```typescript
{
    "title": "Client List",
    "type": "window",
    "name": "clientlist",
    "icon": "http://localhost:22080/resources/icons/client-list.ico",
    "details": {
        "url": "http://localhost:22080/index.html",
        "height": 640,
        "width": 560,
        "left": 100,
        "top": 100,
        "mode": "tab",
        "title": "Client List",
        "backgroundColor": "#1a2b30",
        "focus": false
    }
}
```

- by passing arguments to the `windowsService.open()` call:

```typescript
windowsService.open(
    "clientlist",
    "http://localhost:22080/index.html", 
    {
        height: 640,
        width: 560,
        left: 100,
        top: 100,
        mode: "tab",
        title: "Client List",
        backgroundColor: "#1a2b30",
        focus: false
    })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

## Taskbar Icon

Here is how to setup a taskbar icon:

```typescript
windowsService.open(
    "with-icon",
    "http://theverge.com",
    {
        width: 800,
        height: 600,
        base64ImageSource: "R0lGODlhPQBEAPeoAJosM//AwO/AwHVYZ/z595kzAP/s7P+goOXMv8+fhw/v739/f+8PD98fH/8mJl+fn/9ZWb8/PzWlwv///6wWGbImAPgTEMImIN9gUFCEm/gDALULDN8PAD6atYdCTX9gUNKlj8wZAKUsAOzZz+UMAOsJAP/Z2ccMDA8PD/95eX5NWvsJCOVNQPtfX/8zM8+QePLl38MGBr8JCP+zs9myn/8GBqwpAP/GxgwJCPny78lzYLgjAJ8vAP9fX/+MjMUcAN8zM/9wcM8ZGcATEL+QePdZWf/29uc/P9cmJu9MTDImIN+/r7+/vz8/P8VNQGNugV8AAF9fX8swMNgTAFlDOICAgPNSUnNWSMQ5MBAQEJE3QPIGAM9AQMqGcG9vb6MhJsEdGM8vLx8fH98AANIWAMuQeL8fABkTEPPQ0OM5OSYdGFl5jo+Pj/+pqcsTE78wMFNGQLYmID4dGPvd3UBAQJmTkP+8vH9QUK+vr8ZWSHpzcJMmILdwcLOGcHRQUHxwcK9PT9DQ0O/v70w5MLypoG8wKOuwsP/g4P/Q0IcwKEswKMl8aJ9fX2xjdOtGRs/Pz+Dg4GImIP8gIH0sKEAwKKmTiKZ8aB/f39Wsl+LFt8dgUE9PT5x5aHBwcP+AgP+WltdgYMyZfyywz78AAAAAAAD///8AAP9mZv///wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACH5BAEAAKgALAAAAAA9AEQAAAj/AFEJHEiwoMGDCBMqXMiwocAbBww4nEhxoYkUpzJGrMixogkfGUNqlNixJEIDB0SqHGmyJSojM1bKZOmyop0gM3Oe2liTISKMOoPy7GnwY9CjIYcSRYm0aVKSLmE6nfq05QycVLPuhDrxBlCtYJUqNAq2bNWEBj6ZXRuyxZyDRtqwnXvkhACDV+euTeJm1Ki7A73qNWtFiF+/gA95Gly2CJLDhwEHMOUAAuOpLYDEgBxZ4GRTlC1fDnpkM+fOqD6DDj1aZpITp0dtGCDhr+fVuCu3zlg49ijaokTZTo27uG7Gjn2P+hI8+PDPERoUB318bWbfAJ5sUNFcuGRTYUqV/3ogfXp1rWlMc6awJjiAAd2fm4ogXjz56aypOoIde4OE5u/F9x199dlXnnGiHZWEYbGpsAEA3QXYnHwEFliKAgswgJ8LPeiUXGwedCAKABACCN+EA1pYIIYaFlcDhytd51sGAJbo3onOpajiihlO92KHGaUXGwWjUBChjSPiWJuOO/LYIm4v1tXfE6J4gCSJEZ7YgRYUNrkji9P55sF/ogxw5ZkSqIDaZBV6aSGYq/lGZplndkckZ98xoICbTcIJGQAZcNmdmUc210hs35nCyJ58fgmIKX5RQGOZowxaZwYA+JaoKQwswGijBV4C6SiTUmpphMspJx9unX4KaimjDv9aaXOEBteBqmuuxgEHoLX6Kqx+yXqqBANsgCtit4FWQAEkrNbpq7HSOmtwag5w57GrmlJBASEU18ADjUYb3ADTinIttsgSB1oJFfA63bduimuqKB1keqwUhoCSK374wbujvOSu4QG6UvxBRydcpKsav++Ca6G8A6Pr1x2kVMyHwsVxUALDq/krnrhPSOzXG1lUTIoffqGR7Goi2MAxbv6O2kEG56I7CSlRsEFKFVyovDJoIRTg7sugNRDGqCJzJgcKE0ywc0ELm6KBCCJo8DIPFeCWNGcyqNFE06ToAfV0HBRgxsvLThHn1oddQMrXj5DyAQgjEHSAJMWZwS3HPxT/QMbabI/iBCliMLEJKX2EEkomBAUCxRi42VDADxyTYDVogV+wSChqmKxEKCDAYFDFj4bY7bDGdBhtrnTQYOigeChUmc1K3QTnAUfEgGFgAWt88hKA6aCRIXhxnQ1yg3BCayK44EWdkUQcBByEQChFXfCB776aQsG0BIlQgQgE8qO26X1h8cEUep8ngRBnOy74E9QgRgEAC8SvOfQkh7FDBDmS43PmGoIiKUUEGkMEC/PJHgxw0xH74yx/3XnaYRJgMB8obxQW6kL9QYEJ0FIFgByfIL7/IQAlvQwEpnAC7DtLNJCKUoO/w45c44GwCXiAFB/OXAATQryUxdN4LfFiwgjCNYg+kYMIEFkCKDs6PKAIJouyGWMS1FSKJOMRB/BoIxYJIUXFUxNwoIkEKPAgCBZSQHQ1A2EWDfDEUVLyADj5AChSIQW6gu10bE/JG2VnCZGfo4R4d0sdQoBAHhPjhIB94v/wRoRKQWGRHgrhGSQJxCS+0pCZbEhAAOw=="
    })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

## Subscribing for Window Events

### Opening a Window

You can observe if a new window is opened through `windowsService.windowOpened()` with optional arguments which allow you to filter by name or application. It will emit for every opened window.

Method signature:

```typescript
windowOpened(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
windowsService.windowOpened({ name: "MyWindow" })
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

### Closing a Window

You can observe if a window is closed through `windowsService.windowClosed()` with optional arguments which allow you to filter by name or application. It will emit for every closed window.

Method signature:

```typescript
windowClosed(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
// Notification for every window
windowsService.windowClosed()
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

### Getting Opened Windows

You can get an array of all opened windows by calling `windowsService.windows()` and subscribing to the constructed `Observable<GDWindow[]>`. On subscription, it will emit all opened windows as an array and will emit every time a Glue42 window is opened or closed. You can pass an optional window filter.

```typescript
windows(filterOptions?: WindowFilter): Observable<GDWindow[]>
```

Here is an example how to observe all opened windows associated with the "Client List" application:

```typescript
windowsService.windows({ application: "clientlist" })
    .subscribe({
        next: (openedWindows: GDWindow[]) => {
            // use opened windows here
        }
    });
```
### Getting Focus

You can observe if a window is getting focus through `windowsService.windowGotFocus()` with optional arguments which allow you to filter by name or application. It will emit every time a window gets focus.

Method signature:

```typescript
windowGotFocus(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
// Notification for every window
windowsService.windowGotFocus()
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

### Losing Focus

You can observe if a window is losing focus through `windowsService.windowLostFocus()` with optional arguments which allow you to filter by name or application. It will emit every time a window loses focus.

Method signature:

```typescript
windowLostFocus(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
// Notification for every window
windowsService.windowLostFocus()
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

### Attaching a Tab

You can observe if a tab is attached to a window using `windowsService.tabAttached()`. The returned `Observable<AttachedTab>` will emit for every attached tab.

Method signature:

```typescript
tabAttached(): Observable<AttachedTab>
```

```typescript
windowsService.tabAttached()
    .subscribe({
        next: (attachedTab) => {
            // use tab here
        }
    });
```
### Detaching a Tab

You can observe if a tab is detached from a window using `windowsService.tabDetached()`. The returned `Observable<DetachedTab>` will emit for every detached tab.

Method signature:

```typescript
tabDetached(): Observable<DetachedTab>
```

```typescript
windowsService.tabDetached()
    .subscribe({
        next: (detachedTab) => {
            // use tab here
        }
    });
```

### Changing the Frame Color

You can observe if a window is changing the color of its frame through `windowsService.windowFrameColorChanged()` with optional arguments which allow you to filter by name or application. The returned `Observable<GDWindow>` will emit every time the window changes its frame color.

Method signature:

```typescript
windowFrameColorChanged(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
windowsService.windowFrameColorChanged()
    .subscribe({
        next: (gdWindow) => {
            // use window here
        }
    });
```

### Any Window Event

You can observe any window event by calling `windowsService.event()`. The returned `Observable<GDWindow>` will emit for every window event that occurs.

Method signature:

```typescript
event(filterOptions?: WindowFilter): Observable<GDWindow>
```

```typescript
windowsService.event()
    .subscribe({
        next: (event) => {
            console.log(event)
        }
    });
```

## Group Operations

Windows can form window groups. This can happen through the **Glue42 Enterprise** UI when a user sticks windows together to arrange them on the desktop, or programmatically when you define a window group through the **Window Management** service. The following operations are available for window groups.

### Showing the Header

To show the group header, construct an `Observable<Group>` using the `layoutsService.showHeader()` method. When you subscribe to it, it will show the header of the group and emit the group.

Method signature:

```typescript
showHeader(): Observable<Group>
```

```typescript
windowsService.showHeader()
    .subscribe({
        next: (group) => {
            // use window group here
        }
    });
```

### Hiding the Header

To hide the group header, construct an `Observable<Group>` using the `layoutsService.hideHeader()` method. When you subscribe to it, it will hide the header of the group and emit the group.

Method signature:

```typescript
hideHeader(): Observable<Group>
```

```typescript
windowsService.hideHeader()
    .subscribe({
        next: (group) => {
            // use window group here
        } 
    });
```

### Header Visibility

To track the header visibility, call the `layoutsService.headerVisibilityChanged()` method. The method returns `Observable<boolean>` which emits the header visibility state after every change.

Method signature:

```typescript
headerVisibilityChanged(): Observable<boolean>
```

```typescript
windowsService.headerVisibilityChanged()
    .subscribe({
        next: (headerVisibilityState) => {
            console.log(headerVisibilityState)
        }
    });
```

### Adding Windows

To observe when a new window is added to the group, call the `layoutsService.windowAdded()` method. The method returns `Observable<Group>` which emits the window group after every addition of a window.

Method signature:

```typescript
windowAdded(): Observable<Group>
```

```typescript
windowsService.windowAdded()
    .subscribe({
        next: (group) => {
            // use window group here
        } 
    });
```

### Removing Windows

To observe when a window is removed from the group, call the `layoutsService.windowRemoved()` method. The method returns `Observable<Group>` which emits the window group after every removal of a window.

Method signature:

```typescript
windowRemoved(): Observable<Group>
```

```typescript
windowsService.windowRemoved()
    .subscribe({
        next: (group) => {
            // use window group here
        } 
    });
```

## Window Operations

Once you have a reference to a window instance, you can perform various operations on it. You can control the window bounds, move the window, maximize/minimize/collapse the window, add or remove frame buttons, focus the window, control the window visibility, context, attach tabs, etc.

You can also extract information about the window instance - bounds, context, group ID, window state, visibility, tabs, neighbor windows, etc.

For a full list of the available properties, methods and events applicable to a Glue42 window, see the Glue42 Window reference.

### Accessing the Service

The **GDWindow** service provides methods and properties for a specific window instance and can be accessed by instantiating `GDWindowService` and passing the corresponding window.

```typescript
const myGDWindow = new GDWindowService(myWindowInstance)
```

### My Window Instance

`MyWindowService` is just a "sugar" class providing an easy way to work with the instance of your window. 

An example how to track bounds changes of your window:

```typescript
myWindowService.boundsChanged()
    .pipe(
        map(({ bounds }: GDWindow) => bounds)
    )
    .subscribe({
        next: ({ top, left, width, height }: Bounds) => {
            // use window bounds here
        }
    });
```

### Move and Resize

To move and/or resize a window instance, construct an `Observable<GDWindow>` by using `myWindow.moveResize()`. When you subscribe to it, it will move and/or resize the window instance and emit it. This method accepts the new dimensions/position as an argument.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my())

myGDWindow.moveResize({ 
    width: myWindow.bounds.width + 100,
    top: myWindow.bounds.top + 10
})
.subscribe({
    next: (gdWindow) => {
        // use window here
    }
});
``` 

### Window Context

To update the context of a window instance, construct an `Observable<ContextWrapper>` by using `myWindow.updateContext()`. When you subscribe to it, it will update the context of the window instance and emit an object with the new context and the delta. This method accepts the new context as an argument.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my())
const newContext = { myContext: "changed context" };

myGDWindow.updateContext(newContext).subscribe({
    next: (myWindow) => {
        // use window here
    } 
});
```

### Frame Buttons

To add a button to a window instance frame, construct an `Observable<WindowButton>` by using `myWindow.frameButtonAdded()`. When you subscribe to it, it will create a button and emit the window instance that it belongs to. This method accepts as an argument an object with button information.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my())
const buttonInfo = { buttonId: "search-button", imageBase64: "..." }

myGDWindow.addFrameButton(buttonInfo).subscribe({
    next: (window) => {
        // use window here
    } 
});
```

To remove a button from a window instance frame, construct an `Observable<WindowButton>` by using `myWindow.removeFrameButton()`. When you subscribe to it, it will remove the button and emit the window instance that it belongs. This method accepts a button ID string as an argument.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my());
const buttonId = "search-button";

myGDWindow.removeFrameButton(buttonId).subscribe({
    next: (window) => {
        // use window here
    } 
});
```

### Tabs

To attach a tab to a window instance, construct an `Observable<GDWindow>` by using `myWindow.attachTab()`. When you subscribe to it, it will attach the tab and emit the window instance that it belongs to. This method accepts tab definition (name or GDWindow instance) and index as arguments.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my())
const tab = "myTab";

myGDWindow.attachTab(tab, 0).subscribe({
    next: (window) => {
        // use window here
    } 
});
```

To detach a tab from a window instance, construct an `Observable<GDWindow>` by using `myWindow.detachTab()`. When you subscribe to it, it will detach the tab and emit the window instance that it belongs to. This method accepts an object with options as an argument.

```typescript
const myGDWindow = new GDWindowService(windowService.api.my())
const options = { width: 500 };

myGDWindow.detachTab(options).subscribe({
    next: (window) => {
        // use window here
    } 
});
```

# Layouts

## Accessing the Service

The **Layouts** service can be accessed by injecting `LayoutsService` in any Angular component or service.

*See the Angular [Layouts example](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/layouts) on GitHub.*

```typescript
import { LayoutsService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(layoutsService: LayoutsService) { }
}
```

*If you still need to use the JavaScript [`Layouts API`](../javascript/index.html) directly, you can access it via `layoutsService.api`.*

**Note**: *To use the full capabilities of the `Layouts API` you should enable the Application Manager when configuring Glue42: `appManager: { mode: full }`.*

## Global Layouts

Global saving and restoring is an operation in which all applications running on a user's desktop are saved to a named layout which can later be restored.

### Saving a Global Layout

To save a global layout, construct an `Observable<Layout>` by using `layoutsService.save()`. When you subscribe to it, it will save a new layout and emit it. This method accepts as an argument a configuration object. Note that if a layout with that name already exists, it will be replaced.

```typescript
layoutsService.save({ name: "Client", type: "Global" })
    .subscribe({
        next: (layout) => {
            // use saved layout
        }
    });
``` 

### Restoring a Global Layout

To restore a global layout, construct an `Observable<Layout>` by using the `layoutsService.restore()` method. When you subscribe to it, it will restore an existing layout and emit it.

```typescript
layoutsService.restore({
    name: "Client",
    closeRunningInstance: true
})
.subscribe({
    next: (layout) => {
        // use restored layout
    }
});
``` 

## Activity Layouts

An activity layout is the saved layout of a running activity that can later be restored:

- as a new activity instance - in this case, the restored activity will appear at the exact position the layout was saved;
- or joined to an existing activity instance - in this case, the layout will be arranged around the owner window of the existing activity;

To save an activity layout, call `layoutsService.save()` (same as in Global layouts).

Example:

```typescript
layoutsService.save({ name: "Client", type: "Activity" })
    .subscribe({
        next: (layout) => {
            // use saved layout
        }
    });
``` 

To restore an activity layout, call `layoutsService.restore()` (same as in Global layouts).

```typescript
layoutsService.restore({
    name: "Client",
    type: "Activity",
    closeRunningInstance: true
})
.subscribe({
    next: (layout) => {
        // use restored layout
    }
});
``` 

## Managing Layouts

### Removing Layouts

To remove a layout, construct an `Observable<void>` by using the `layoutsService.remove()` method. When you subscribe to it, it will remove an existing layout. You must pass the type of the layout and its name.

```typescript
layoutsService.remove("Global", "Client")
    .subscribe({
        next: () => console.log("Layout removed")
    });
```

### Renaming Layouts

To rename a layout, construct an `Observable<void>` by using the `layoutsService.rename()` method. When you subscribe to it, it will rename an existing layout. You must pass the layout and the new name as arguments.

```typescript
layoutsService.rename(existingLayout, "Client")
    .subscribe({
        next: () => console.log("Layout renamed")
    });
```

### Updating Layout Metadata

To update the metadata of a layout, construct an `Observable<void>` by using the `layoutsService.updateMetadata()` method. When you subscribe to it, it will update the metadata of an existing layout. You must pass the layout as an argument.

```typescript
layoutsService.updateMetadata(existingLayout)
    .subscribe({
        next: () => console.log("Layout metadata updated")
    });
```

### Exporting Layouts

To export all saved layouts, construct an `Observable<Layout[]>` by calling the `layoutsService.export()` method, which on subscribe will emit all layouts in the system. Exporting layouts can be useful if you want to store them on a database and then use them as restore points (if the user for some reason wants to). Exported layouts can also be sent to another user and imported on their machine.

Example:

```typescript
layoutsService.export()
    .subscribe({
        next: (layouts) => {
            // use exported layouts
        }
    });
```

Exporting is different from using the [`layoutsService.api.list()`](../../../../reference/glue/latest/layouts/index.html#API-list) method, as it does not return only the application default layouts, but all layouts in the system.

### Importing Layouts

To import layouts, construct an `Observable<void>` by calling the `layoutsService.import()` method. It accepts an array of `Layout` objects and constructs an `Observable`, which imports the layouts from the array when you subscribe to it.

```javascript
const mode = "replace";
layoutsService.import(layouts, mode)
    .subscribe({
        next: () => {
            // do something
        }
    });
```

The [`ImportMode`](../../../../reference/glue/latest/layouts/index.html#ImportMode) controls the import behavior. If the mode is `replace` (the default value), all existing layouts will be removed. If it is `merge`, the layouts will be imported and merged with the existing layouts.

## Subscribing for Layout Updates

To retrieve all saved layouts, call `layoutsService.layouts()`. The method returns an `Observable<Layout[]>`, which on subscribe will emit all saved layouts as an array, and will emit every time a layout is added, changed or removed. You can also pass an optional `LayoutFilter` object to target specific layouts.

Here is an example where you can observe all `Global` layouts:

```typescript
layoutsService.layouts({ type: "Global" })
    .subscribe({
        next: (globalLayouts: Layout[]) => { 
            // use global layouts
        }
    });
```

To subscribe for new layouts or updates of existing layouts, call `layoutsService.layoutSaved()`. The method returns `Observable<Layout>`. You can also pass an optional `LayoutFilter` object. If a filter is provided and all its options are set (`type` and `name`) and it matches an already existing layout, this layout will be emitted immediately.

```typescript
layoutsService.layoutSaved({
    type: "Global",
    name: "Client"
})
.subscribe({
    next: (layout) => {
        // use layout
    }
});
```

To subscribe for removing layouts, call `layoutsService.layoutRemoved()` with filter options object. The method returns `Observable<Layout>`. If the filter options object is not provided it will emit for every removed layout.

```typescript
layoutsService.layoutRemoved({
    type: "Global",
    name: "Client"
})
.subscribe({
    next: (layout) => {
        // use layout
    }
});
```

# Notifications

## Architecture

![GNS Architecture](../../../images/notifications/gns.gif)

## Accessing the Service

The **Notifications** service can be accessed by injecting `NotificationsService` in any Angular component or service.

```typescript
import { NotificationsService } from "@glue42/ng-glue";

@Component({
  selector: "app-root"
  templateUrl: "app.component.html"
})
export class AppComponent {
  constructor(notificationsService: NotificationsService) { }
}
```

## Raising Notifications

To raise a GNS notification using the **Notifications** service, call `notificationsService.raiseNotification()` and subscribe to the returned `Observable<void>` to execute the notification.

Method signature:

```typescript
raiseNotification(notification: GnsNotificationSettings): Observable<void>
```

Example:

```typescript
notificationsService.raiseNotification({
    title: "Critical Alert",
    severity: "High",
    description: "Your computer will be restarted in 30 seconds"
})
.subscribe({
    next: () => console.log("Raised a notification"),
    error: console.error
});
```

![Raising Notifications](../../../images/notifications/js-raising-notification.png)

*See the Angular [Notifications example](https://github.com/Glue42/ng-glue-examples/tree/master/projects/examples/src/app/notifications) on GitHub.*

## Subscribing for Notifications

The **Notifications** service provides a property `notification: Observable<GnsNotification>` to which you can subscribe in order to be notified when a GNS notification has been raised.

Example:

```typescript
notificationsService.notification
    .subscribe({
        next: ({ title, creationTime }) => {
            console.log(`Notification ${title} created at ${creationTime}`)
        }
    });
```