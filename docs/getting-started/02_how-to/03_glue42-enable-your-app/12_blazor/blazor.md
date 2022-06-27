## Overview

All [Glue42 .NET](../net/index.html) functionalities are available for your Blazor WebAssembly and Blazor Server apps through the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library.

Your Glue42 enabled Blazor apps can be configured as Glue42 apps (see [App Configuration](#app_configuration)) in order to be started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) and hosted in [Glue42 Windows](../../../../glue42-concepts/windows/window-management/overview/index.html), or they can run independently in a browser.

*For the differences in initializing the Glue42 library in hosted and browser Blazor apps, see the [Hosted & Browser Apps](#initialization-hosted__browser_apps) section).*

Blazor apps started by [**Glue42 Enterprise**](https://glue42.com/enterprise/) will be hosted in [Glue42 Windows](../../../../glue42-concepts/windows/window-management/overview/index.html) enabling them to stick to other Glue42 Windows, to use [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/overview/index.html), and to be saved and restored in [Layouts](../../../../glue42-concepts/windows/layouts/overview/index.html).

### Blazor WebAssembly

The connection to Glue42 in a Blazor WebAssembly app originates from the client app (the browser webpage) which means that the Blazor WebAssembly app will behave as a desktop app and will connect to the locally installed [**Glue42 Enterprise**](https://glue42.com/enterprise/).

*See the [Blazor WebAssembly example](https://github.com/Glue42/net-examples/tree/master/glazor/GlazorWebAssembly) on GitHub which demonstrates the various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

### Blazor Server

If you plan on using the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library in Blazor Server apps, note that the connection to Glue42 will originate from the server side. This means that you can either install the Glue42 Gateway on a visible location (cloud), or use the Blazor server on the desktop machine (e.g., Docker, IIS) where you have a running [**Glue42 Enterprise**](https://glue42.com/enterprise/). For Blazor Server apps, you can integrate the front-end with the [Glue42 JavaScript](../javascript/index.html) library.

## Referencing

The [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library is available as a NuGet package which you can include and configure in your projects. Download and reference the latest [`GlueBase`](https://www.nuget.org/packages/GlueBase/) version.

*See the [Blazor examples](https://github.com/Glue42/net-examples/tree/master/glazor) on GitHub demonstrating the various [**Glue42 Enterprise**](https://glue42.com/enterprise/) features.*

## Initialization

To initialize the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library, you can use the [`GlueProvider`](https://github.com/Glue42/net-examples/blob/master/glazor/GlazorWebAssembly/GlazorWebAssembly/GlueProvider.cs) demo class from the [Blazor examples](https://github.com/Glue42/net-examples/tree/master/glazor) that demonstrates all necessary initialization logic and can be modified or used as is, per your needs.

*For more in-depth examples of initializing the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library, see the [Hosted & Browser Apps](#initialization-hosted__browser_apps) section.*

The [`GlueProvider`](https://github.com/Glue42/net-examples/blob/master/glazor/GlazorWebAssembly/GlazorWebAssembly/GlueProvider.cs) class supports:

- Glue42 initialization for Blazor WebAssembly and Blazor Server apps hosted in [**Glue42 Enterprise**](https://glue42.com/enterprise/).
- Glue42 initialization for Blazor WebAssembly and Blazor Server apps opened in a browser.
- Blazor logging facade.

The following example demonstrates how to plug in the `GlueProvider` class in your Blazor app:

```csharp
// Bridging the logger.
builder.Services.AddScoped<IGlueLoggerFactory, GlueLoggerFactory>(serviceProvider =>
                new GlueLoggerFactory(serviceProvider.GetService<ILoggerFactory>()));

// Plugging in the `GlueProvider` class.
builder.Services.AddScoped(typeof(GlueProvider));
```

After that, you can use Glue42 in your RAZOR file:

Injection:

```csharp
@inject IGlueLoggerFactory GlueLoggerFactory
@inject GlueProvider glueProvider
```

Initialization:

```csharp
@code {
   private Task<IGlue42Base> GetGlue()
        => glueProvider.InitGlue();
}
```

The following example demonstrates how to use the Glue42 APIs after the initialization of the library:

```csharp
// Getting the Glue42 object - entry point for all Glue42 APIs.
var glue = await GetGlue().ConfigureAwait(false);

// Using the Channels API to join a Channel and update its context.
var redChannel = await glue.Channels
    .AwaitChannel(channel => channel.Name == "Red").ConfigureAwait(false);

context = glue.Channels.JoinChannel(redChannel);

await context.SetValue(ric, "partyPortfolio.ric").ConfigureAwait(false);
```

*All concepts are demonstrated in the [Blazor examples](https://github.com/Glue42/net-examples/blob/master/glazor/GlazorWebAssembly/GlazorWebAssembly/Pages/Glue42.razor).*

### Hosted & Browser Apps

When initializing the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library in Blazor apps, you have to consider whether your app is configured as a Glue42 app and can be started by [**Glue42 Enterprise**](https://glue42.com/enterprise/), or is running independently in a browser. In the different scenarios you will need different mechanisms for providing authentication and app information to Glue42. In the case of a hosted Blazor app, [**Glue42 Enterprise**](https://glue42.com/enterprise/) injects this information in the Glue42 Window and you must expose a function to extract it. In the case of a Blazor app running in a browser, this information must be provided manually through user input.

#### Hosted Apps

If your Blazor WebAssembly or Blazor Server app is hosted in [**Glue42 Enterprise**](https://glue42.com/enterprise/), you must expose a JavaScript function that will pull the necessary information from the Glue42 Window and then initialize the Glue42 library with it.

*See a demo implementation of the exposed JavaScript function in the [Blazor WebAssembly example](https://github.com/Glue42/net-examples/blob/master/glazor/GlazorWebAssembly/GlazorWebAssembly/wwwroot/js/gd.js).*

The following example demonstrates how to invoke the exposed function:

```csharp
initOptions = await Glue42Base.GetHostedGDOptions(
    async tokenName => await jsRuntime_.InvokeAsync<string>(tokenName).ConfigureAwait(false),
    async gdInfoPropName =>
    {
        var gdHostInfo = await GetJSProp<GDHostInfo>(gdInfoPropName).ConfigureAwait(false);
        windowId = gdHostInfo.WindowId;
        return gdHostInfo;
    }).ConfigureAwait(false);
```

#### Browser Apps

If your Blazor app is opened in a browser window, you must provide the username, the Glue42 authentication details and the app name yourself:

```csharp
var username = await GetPromptInput("user name").ConfigureAwait(false);
var appName = await GetPromptInput("app name").ConfigureAwait(false);

initOptions = new InitializeOptions
{
    AdvancedOptions = new AdvancedOptions
    {
        AuthenticationProvider = new GatewaySecretAuthenticationProvider(username, username)
    },
    // Make sure that the app name is different for each scoped `GlueProvider`.
    ApplicationName = appName
};
```

#### Initialization

When you have built the initialization options using either mechanism, initialize the Glue42 library to obtain the Glue42 object - the entry point for all Glue42 APIs:

```csharp
var glue = await Glue42Base.InitializeGlue(initOptions).ConfigureAwait(false);
```

### Socket Implementation

For Blazor WebAssembly apps, you have to select an appropriate socket implementation. Otherwise, it isn't necessary to change the socket implementation:

```csharp
// Choosing socket implementation appropriate for Blazor WebAssembly apps.
initOptions.AdvancedOptions.SocketFactory = connection =>
    new ClientSocket(new Uri(initOptions.GatewayUri ?? DefaultGatewayUri), new Configuration());
```

### Logging

You can choose your own logging facade and then create a bridge so that Glue42 will log through your logging mechanism.

*See the demo logging implementation in the [Blazor WebAssembly example](https://github.com/Glue42/net-examples/blob/master/glazor/GlazorWebAssembly/GlazorWebAssembly/GlueLoggerFactory.cs).*

Implement the bridging contract and instruct Glue42 to use your logging mechanism:

```csharp
initOptions.LoggerFactory = your_logging_factory;
```

The [Blazor examples](https://github.com/Glue42/net-examples/blob/master/glazor/) use the Microsoft logging mechanism plugged in the following way:

```csharp
builder.Services.AddScoped<IGlueLoggerFactory, GlueLoggerFactory>(serviceProvider =>
    new GlueLoggerFactory(serviceProvider.GetService<ILoggerFactory>()));
```

## App Configuration

To add your Blazor app to the [Glue42 Toolbar](../../../../glue42-concepts/glue42-toolbar/index.html), you must create a JSON file with app configuration. Place this file in the `%LocalAppData%\Tick42\UserData\<ENV-REG>\apps` folder, where `<ENV-REG>` must be replaced with the environment and region of your [**Glue42 Enterprise**](https://glue42.com/enterprise/) copy (e.g., T42-DEMO).

The following is an example configuration for a Blazor app:

```json
{
    "name": "my-app",
    "title": "My App",
    "type": "window",
    "details": {
        "url": "https://example.com/my-app/",
        "mode": "tab",
        "width": 500,
        "height": 400
    }
}
```

The `"name"`, `"type"` and `"url"` properties are required and `"type"` must be set to `"window"`. The `"url"` property points to the location of the web app.

For more details, see the [Developers > Configuration > Application](../../../../developers/configuration/application/index.html#app_configuration-window) section.

## Glue42 .NET Concepts

Once the [`GlueBase`](https://www.nuget.org/packages/GlueBase/) library has been initialized, your app has access to all Glue42 functionalities. For more detailed information on the different Glue42 concepts and APIs, see:

- [App Management](../../../../glue42-concepts/application-management/net/index.html)
- [Intents](../../../../glue42-concepts/intents/net/index.html)
- [Shared Contexts](../../../../glue42-concepts/data-sharing-between-apps/shared-contexts/net/index.html)
- [Channels](../../../../glue42-concepts/data-sharing-between-apps/channels/net/index.html)
- [Interop](../../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html)
- [Pub/Sub](../../../../glue42-concepts/data-sharing-between-apps/pub-sub/net/index.html)
- [Window Management](../../../../glue42-concepts/windows/window-management/net/index.html)
- [Layouts](../../../../glue42-concepts/windows/layouts/net/index.html)
- [Notifications](../../../../glue42-concepts/notifications/net/index.html)
- [Metrics](../../../../glue42-concepts/metrics/net/index.html)