## Overview

The Glue42 Angular wrapper, [`@glue42/ng`](https://www.npmjs.com/package/@glue42/ng), provides you with an easy way to initialize the Glue42 JavaScript libraries and use Glue42 functionalities in your projects. It works with the [@glue42/web](https://www.npmjs.com/package/@glue42/web) and [@glue42/web-platform](https://www.npmjs.com/package/@glue42/web-platform) libraries, if you are working on a [**Glue42 Core**](https://glue42.com/core/) project, and with the [@glue42/desktop](https://www.npmjs.com/package/@glue42/desktop) library, if you are working on a [**Glue42 Enterprise**](https://glue42.com/enterprise/) project. The examples below use the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript library](../../../../reference/glue/latest/glue/index.html).

### Legacy Angular Wrapper

If you are using the legacy Glue42 Angular wrapper, [`@glue42/ng-glue`](https://www.npmjs.com/package/@glue42/ng-glue), you can download the documentation for it from [here](../../../../assets/glue42-angular-legacy.md). The documentation on this site is relevant only to the new light-weight Glue42 Angular wrapper - [`@glue42/ng`](https://www.npmjs.com/package/@glue42/ng).

## Prerequisites

This package should be used only in Angular applications. If your app was created with the Angular CLI, then you don't need to take any additional steps. Otherwise, make sure to install the peer dependencies of `@glue42/ng`:

```json
{
    "dependencies": {
        "@angular/common": "^9.1.3",
        "@angular/core": "^9.1.3",
        "rxjs": "^6.5.5",
        "tslib": "^1.10.0"
    }
}
```

*Note that [`@glue42/ng`](https://www.npmjs.com/package/@glue42/ng) supports Angular 7+.*

The example below assumes that your app was created with the Angular CLI. Install the `@glue42/ng` library:

```cmd
npm install --save @glue42/ng
```

## Library Features

The Glue42 Angular library exposes two important elements:
- `Glue42Ng` - an Angular module that initializes the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript library](../../../../reference/glue/latest/glue/index.html);
- `Glue42Store` - an Angular service that gives access to the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript API;

### Glue42Ng Module

The `Glue42Ng` module is responsible for initializing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript library](../../../../reference/glue/latest/glue/index.html). You must import the `Glue42Ng` module *once* for the entire application - in the *root module* by using the `forRoot()` method. This methods accepts a settings object which has the following signature:

```typescript
export interface Glue42NgSettings {
    holdInit?: boolean;
    web?: {
        factory?: Glue42WebFactoryFunction;
        config?: Glue42Web.Config;
    };
    webPlatform?: {
        factory?: Glue42WebPlatformFactoryFunction;
        config?: Glue42WebPlatform.Config;
    };
    desktop?: {
        factory?: Glue42DesktopFactoryFunction;
        config?: Glue42.Config;
    };
}
```

The table below describes the properties of the `Glue42NgSettings` object.

| Property | Description |
|----------|-------------|
| `web` | *Optional*. An object with two properties: `config` and `factory`. The `config` property accepts a configuration object for the [Glue42 Web](https://www.npmjs.com/package/@glue42/web) library. The `factory` property accepts the factory function exposed by Glue42 Web. You should define this object if your application is a Web Client. |
| `webPlatform` | *Optional*. An object with two properties: `config` and `factory`. The `config` property accepts a configuration object for the [Web Platform](https://www.npmjs.com/package/@glue42/web-platform) library. The `factory` property accepts the factory function exposed by Glue42 Web Platform. You should define this object if your application is a Web Platform application (or Main application) in the context of [**Glue42 Core**](https://glue42.com/core/). |
| `desktop` | *Optional*. An object with two properties: `config` and `factory`. The `config` property accepts a configuration object for the [@glue42/desktop](https://www.npmjs.com/package/@glue42/desktop) library used in [**Glue42 Enterprise**](https://glue42.com/enterprise/). The `factory` property accepts the factory function exposed by the library. You should define this object if your application is a [**Glue42 Enterprise**](https://glue42.com/enterprise/) application. |
| `holdInit` | *Optional*. Defines whether your app initialization must wait for the Glue42 factory function to resolve. Defaults to `true`. |

*Note that you can define either the `web`, or the `webPlatform` property together with `desktop`. This is useful if you want your application to have different initialization characteristics in [**Glue42 Core**](https://glue42.com/core/) and [**Glue42 Enterprise**](https://glue42.com/enterprise/).*

All properties are optional, but it is recommended that you provide the factory functions explicitly. If no factory functions are provided, the library will try to select an appropriate function attached to the global `window` object.

The initialization of the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library is asynchronous and therefore can take anywhere between a few milliseconds and a couple of seconds. There are two main situations in which setting `holdInit` to `true` (default) or `false` will benefit your project:

- `holdInit: false` - If the Glue42 functionalities play only a supporting role in your project, rather than being an essential part of it, it is recommended that you set `holdInit` to `false`. This way, your app won't have to wait for the Glue42 library to initialize in order to be able to function properly. You can use the `Glue42Store` service to get notified when the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library is ready.

- `holdInit: true` - If the Glue42 functionalities, however, are a critical part your project, then it is recommended to leave `holdInit` set to `true`. This way, Angular will wait for the Glue42 factory function to resolve before bootstrapping your first component. This will spare you the need to check whether the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library is available or not every time you want to use it in your app. As a negative result to this approach, when your users load the app, they will keep seeing a blank screen up until the first component has been bootstrapped. Of course, you can solve this by providing a loader animation as soon as your app is accessed.

The example below shows how to initialize the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library by passing a factory function and a custom configuration object:

```javascript
import { Glue42Ng } from "@glue42/ng";
import Glue from "@glue42/desktop";

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        Glue42Ng.forRoot({ desktop: { factory: Glue, config: { channels: true } } })
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

*It is important to note that if the Glue42 initialization fails for any reason (invalid configuration, missing factory function, connection problems or initialization timeout), your app will still initialize.*

### Glue42Store Service

The `Glue42Store` service is used to obtain the `glue` object which exposes the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript API](../../../../reference/glue/latest/glue/index.html). This service can also notify you when the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library has been initialized and enables you to check for any initialization errors.

Example of creating a `Glue42Store` service:

```javascript
import { Injectable } from "@angular/core";
import { Glue42Store } from "@glue42/ng";

@Injectable()
export class Glue42Service {
    constructor(private readonly glueStore: Glue42Store) { }
}
```

The `Glue42Store` service offers the following methods:

| Method | Description |
|--------|-------------|
| `this.glueStore.ready()` | Returns an `Observable`. Subscribe to it to get notified when the [**Glue42 Enterprise**](https://glue42.com/enterprise/) library has been initialized. If the initialization fails, you will receive an object with an `error` property, otherwise the object will be empty. This is particularly useful if you set `holdInit` to `false` when initializing the library, because you need to make sure that the Glue42 library is ready for use before accessing any of its APIs. |
| `this.glueStore.getInitError()` | Returns an initialization error object from the Glue42 factory function or `undefined`. |
| `this.glueStore.getGlue()` | Returns the [**Glue42 Enterprise**](https://glue42.com/enterprise/) API object. If needed, it is up to you to cast the returned object to the respective type (either `Glue42.Glue` or `Glue42Web.API` depending on the used Glue42 JavaScript library). |

You can now inject the service in the components that need it and access the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript API](../../../../reference/glue/latest/glue/index.html) from the object returned by `this.glueStore.getGlue()`. This gives you a decent level of encapsulation and control. If you prefer handling async actions with an `Observable`, then this service is the perfect place to wrap in an `Observable` the method you want to use.

## Usage

Below you can see some examples of initializing and using the Glue42 Angular library.

### Initialization

Import the `Glue42Ng` module in the root module of your app and pass the factory function from `@glue42/desktop`:

```javascript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";

import { AppComponent } from "./app.component";
import { Glue42Ng } from "@glue42/ng";
import Glue from "@glue42/desktop";

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        Glue42Ng.forRoot({ dekstop: { factory: Glue } })
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

### Consuming Glue42 Enterprise APIs

Inject the `Glue42Store` service in your component/service of choice in order to use the [**Glue42 Enterprise**](https://glue42.com/enterprise/) [JavaScript API](../../../../reference/glue/latest/glue/index.html). It is recommended that you create your own Angular service that injects the `Glue42Store` and exposes only the functionality your app needs.

When initializing the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library with the `Glue42Ng` module, you can use the `holdInit` property (see [Glue42Ng Module](#library_features-glue42ng_module)) to configure whether the Angular framework must wait for the Glue42 factory function to resolve before bootstrapping your first component. Depending on this setting, you can use the `Glue42Store` service in different ways. Below are given examples and short explanations for both cases:

- #### holdInit: true

Creating the service:

```javascript
import { Injectable } from "@angular/core";
import { Glue42Store } from "@glue42/ng";

@Injectable()
export class Glue42Service {

    constructor(private readonly glueStore: Glue42Store) { }

    public get glueAvailable() {
        return !this.glueStore.getInitError();
    }

    public registerMethod(name: string, callback: () => void): Promise<void> {
        if (!this.glueAvailable) {
            return Promise.reject("Glue42 wasn't initialized.");
        }
        return this.glueStore.getGlue().interop.register(name, callback);
    }
}
```

Using the service:

```javascript
import { Component, OnInit } from "@angular/core";
import { Glue42Service } from "./my-glue-service";

@Component({
    selector: "app-root",
    templateUrl: "./app.component.html",
    styleUrls: ["./app.component.css"]
})
export class AppComponent implements OnInit {

    constructor(private glueService: Glue42Service) { }

    public ngOnInit(): void {
        if (!this.glueService.glueAvailable) {
            // Тhere has been an error during the Glue42 initialization.
            return;
        }
        // Glue42 has been initialized without errors and is ready to use.
        this.glueService.registerMethod("MyMethod", () => console.log("Doing work!"));
    }
}
```

If you set `holdInit` to `true` (default), you can be sure that everywhere you inject the `Glue42Store` service, the respective properties will be initialized and set. This is very convenient, because you don't have to subscribe and wait for an event in order to use the [**Glue42 Enterprise**](https://glue42.com/enterprise/) JavaScript library. However, you do need to always check if there is an initialization error by using `this.glueStore.getInitError()`. If the Glue42 factory functions rejects or throws an error, your app won't crash, but the Glue42 library won't be available and the value returned by `getInitError()` will be set to the respective error object during initialization.

- #### holdInit: false

Creating the service:

```javascript
import { Injectable } from "@angular/core";
import { Glue42Store } from "@glue42/ng";

@Injectable()
export class Glue42Service {

    constructor(private readonly glueStore: Glue42Store) { }

    public ready() {
        return this.glueStore.ready;
    }

    public registerMethod(name: string, callback: () => void): Promise<void> {
        return this.glueStore.getGlue().interop.register(name, callback);
    }
}
```

Using the service:

```javascript
import { Component, OnInit } from "@angular/core";
import { Glue42Service } from "./my-glue-service";

@Component({
    selector: "app-root",
    templateUrl: "./app.component.html",
    styleUrls: ["./app.component.css"]
})
export class AppComponent implements OnInit {

    constructor(private glueService: Glue42Service) { }

    public ngOnInit(): void {
        // Show the loader.
        this.glueService
            .ready()
            .subscribe((glueStatus) => {
                if (glueStatus.error) {
                    // Hide the loader.
                    // Glue42 isn't available.
                    return;
                }
                // Hide the loader.
                // Glue42 is ready, continue with your logic.
                this.glueService.registerMethod("MyMethod", () => console.log("Doing work!"));
            })
    }
}
```

As you can see, this approach requires a little bit more code, but it gives you an easy way to provide pleasant user experience while Glue42 is initializing, handle gracefully any initialization errors, and when the initialization resolves normally, you don't need to check the error object as in the previous example.

## Glue42 JavaScript Concepts

Once the Glue42 Angular library has been initialized, your application has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [Application Management](../../../../glue42-concepts/application-management/javascript/index.html)
- [Application Preferences](../../../../glue42-concepts/application-preferences/javascript/index.html)
- [Intents](../../../../glue42-concepts/intents/javascript/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/javascript/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/javascript/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/javascript/index.html)
- [Pub/Sub](../../../../glue42-concepts/data-sharing-between-apps/pub-sub/javascript/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/javascript/index.html)
- [Workspaces](../../../../glue42-concepts/windows/workspaces/javascript/index.html)
- [Layouts](../../../../glue42-concepts/windows/layouts/javascript/index.html)
- [Notifications](../../../../glue42-concepts/notifications/javascript/index.html)
- [Global Search](../../../../glue42-concepts/global-search/index.html)
- [Metrics](../../../../glue42-concepts/metrics/javascript/index.html)
- [Hotkeys](../../../../glue42-concepts/glue42-platform-features/index.html#hotkeys)
- [Zooming](../../../../glue42-concepts/glue42-platform-features/index.html#zooming)
- [Displays](../../../../glue42-concepts/glue42-platform-features/index.html#displays)
- [Logging](../../../../glue42-concepts/glue42-platform-features/index.html#logging)

## Reference

For a complete list of the available JavaScript APIs, see the [Glue42 JavaScript Reference Documentation](../../../../reference/glue/latest/glue/index.html).