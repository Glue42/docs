## Overview

The purpose of this tutorial is to introduce [**Glue42 Enterprise**](https://glue42.com/enterprise/) and some of its features to .NET developers and to show the Glue42 way of developing apps. The tutorial starts with four WPF apps working independently from each other and when completed, the same apps will be integrated with [**Glue42 Enterprise**](https://glue42.com/enterprise/) and will interoperate as one unified user-friendly system.

## Prerequisites

- [**Glue42 Enterprise**](https://glue42.com/enterprise/) installed and running.
- Experience with C# and WPF.
- Basic understanding of asynchronous programming.

## Resources

Before you start the tutorial, see the [.NET examples](https://github.com/Glue42/net-examples) on GitHub - a repository with sample projects that use the Glue42 .NET API.

## Setup

First, clone the [tutorial repository](https://github.com/Glue42/net-tutorial)  and start [**Glue42 Enterprise**](https://glue42.com/enterprise/).

After that, reference the `Glue42.dll`, located in `%LocalAppData%/Tick42/GlueSDK/Glue42NET/lib/net45`, in each of the projects.

Repository structure:
- `data` - contains an EXE which hosts all client related data on port 8083;
- `config` - contains the configuration files for all apps;
- `skeleton` - contains the skeleton of the solution in which you will start;
- `solution` - contains a full solution to the tutorial, which you can use for reference how everything should work when completed;
- `start` - the folder in which your work will begin;

The tutorial contains the following apps:
- **Clients** - WPF app which has information about clients and visualizes a grid with very little details;
- **Contacts** - WPF app which has a grid with contacts;
- **Portfolio** - WPF app visualizing a client's profile in detail;
- **Notifications** - WPF app which can raise notifications by clicking a button;

So, now you have four apps that must be integrated with [**Glue42 Enterprise**](https://glue42.com/enterprise/) in order to maximize the working efficiency and to improve the user experience. We can start with copying the skeleton projects from `/skeleton` to `/start`. Then, in the configuration folder, you will find JSON files which are the pre-made configuration files for your apps. To start your apps from Glue42 Enterprise, you should update the `path` properties (they must point to the location of the app on your machine) in all of them and copy the JSON files in the app configuration folder (`%LocalAppData%/Tick42/GlueDesktop/config/apps`). Note that the folder is monitored at runtime, so your apps will appear instantly. Now, you can start all apps through the Glue42 Toolbar. In the Glue42 Toolbar, all apps will have `Tutorial` prefixed to their titles to make them easier to find.

Here are some common issues that you may encounter:
- App is missing from the Glue42 Toolbar - check its configuration file: it must be a valid `JSON`.
- App doesn't start after being clicked - make sure that the `path` property of the configuration file of the app points to the folder which contains the EXE.
- The `Clients` app doesn't visualize any data - the data provider EXE must be running, so make sure that the `path` in its configuration file is valid and that port 8083 is free (this is where the data is hosted).
- Some/All of my projects won't build - make sure that all projects reference the `Glue42.dll` located in `%LocalAppData%/Tick42/GlueSDK/Glue42NET/lib/net45`.

## Initializing the Glue42 Library

To use Glue42 library in your .NET app, you must first initialize it. This is accomplished by invoking the `Initialize` method of a `Glue42` object:

```csharp
// You can find similar initialization in the App class of all apps
var appName = "SampleName";
var glue = new Glue42();
glue.Initialize(appName);
```

It is best that this happens at the first possible moment so that Glue42 is always available. In all apps, the initialization is done `OnStartup` in the `App.xaml.cs` files. In most cases, it will be enough to just pass your app name in the Glue42 initialization method, however if you want better performance or to tinker with the advanced options you can check out the [Glue42 Enable Your App](../../../getting-started/how-to/glue42-enable-your-app/net/index.html) documentation.

## 1. Window Management

Users of desktop apps start to face issues with window management when they have to work with many apps and their desktop gets cluttered with random windows. Glue42 has a solution for this - the StickyWindows API. With this API you can make your app sticky which gives your users the opportunity to organize their desktops. To make the app windows sticky, find all `RegisterToStickyWindows()` methods in your apps and implemented them so that all apps are flat windows and their titles are the same as their names. You can use the given bounds objects in the configuration. For more details on how to do this, see the .NET [Window Management](../../../glue42-concepts/windows/window-management/net/index.html) documentation.

*For each task you will find hints and guidelines as comments in the code.*

![Glue .Net Tutorial Chapter 1](../../../images/tutorials/enterprise-net/1-net.gif "Glue .Net Tutorial Chapter 1")

## 2. Shared Contexts

Glue42 offers many ways to share data between apps, the easiest one being the [Shared Contexts](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/net/index.html) API. The Shared Contexts API can pass data globally to all currently running apps. This data can be updated by everyone and everyone can listen for the updates.

First, we will start from the `Clients` project. In the code you can find the `ClientList_MouseDoubleClick()` method. The `CurrentParty` context in it should be updated, so that every time the user clicks on a row, the context is updated. The `Portfolio` app should listen for changes and display the data. You should now follow the instructions in the code and implement this functionality. More information about the Shared Contexts API can be found [here](../../../glue42-concepts/data-sharing-between-apps/shared-contexts/net/index.html).

*Note that despite not being required, the instructions suggest that you should strongly type your context with an interface instead of using it as a `Dictionary<string,object>`.*

![Glue .Net Tutorial Chapter 2](../../../images/tutorials/enterprise-net/2-net.gif)

## 3. Interop - Declarative Model

The Glue42 .NET API has two Interop APIs - [declarative](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html) and [imperative](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html#imperative_model). This tutorial covers only the declarative Interop model. If you want to use the imperative API, see [here](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html#imperative_model).

### 3.1 Method Registration

The [Interop](../../../glue42-concepts/data-sharing-between-apps/interop/overview/index.html) API is another way through which you can share data with other apps. In this case, it is a better form of communication, so you should comment out your context code. The `Portfolio` app is responsible for visualizing data, so it should register a method `PopulatedData()` which accepts the needed data and updates the `Portfolio` UI. You can find more information on how to register a method [here](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html#declarative_model).

### 3.2 Method Invocation

The `Clients` app has data for visualization so it must invoke `PopulateData()` on every row click (don't forget to check if the service exists, because a row might be clicked when `Portfolio` is closed). When you implement this, you will see the same result as the one from the Shared Contexts. You can find more information on how to invoke a method [here](../../../glue42-concepts/data-sharing-between-apps/interop/net/index.html#declarative_model).

## 4. Notifications

### 4.1 Raising Notifications

Glue42 has a notification API which is accessible through `Glue.Notifications`. It can publish notifications with information and possible actions on them in the form of toasts which appear in the bottom right corner of the screen. Your next task is to implement the `RaiseButton_Click()` method in the `Notifications` project. When you are done, the raise button should raise a basic notification with the same title as the text in the text box of the app. For more information on notifications, [see here](../../../glue42-concepts/notifications/net/index.html).

### 4.2 Glue42 Routing

If you click on a notification, a default screen will be opened with further information. Nevertheless, Glue42 can be configured so that a custom Interop method will be executed when a notification is clicked. The name of the method is passed in the notification constructor. You must register a service in the `Clients` app which has a method that opens the `Contacts` app. For more information on Glue42 routing, [see here](../../../glue42-concepts/notifications/net/index.html). To see how to open an app, continue to the next chapter.

## 5. App Management

Glue42 has an App Management API through which you can execute all kinds of app management commands. For the current scenario you will only need to start a new instance of the `Contacts` app in the `ShowContacts()` method. To see how to use the App Management API, click [here](../../../glue42-concepts/application-management/net/index.html). Keep in mind that the `Clients` app must be open when you click on a notification, if you want the `ShowContacts()` method to be invoked.

So, now the `Contacts` app is opened, but you still need to have data in it. You can get your data from the `Clients` app. Go to the `ContactsService` class in `Clients`, implement the `FindWhoToCall()` method, invoke it from `Contacts` and use the result to update the UI. Don't forget to check if the service exists.

![Glue .Net Tutorial Chapter 5](../../../images/tutorials/enterprise-net/5-net.gif)

## 6. Metrics

The [Metrics](../../../glue42-concepts/metrics/net/index.html) API of Glue42 is used to track information provided from any app. You can record the current state of the app, the current user, is the app running, what is its CPU usage and more.

Your task will be to add metrics in the `DataReceiver` class in the `Clients` project. There are no restrictions to what you can measure, but it makes most sense to log the URL that is used to contact the server, the time it takes the server to respond and log any exceptions thrown from the request. To see more about the Metrics API, click [here](../../../glue42-concepts/metrics/net/index.html).

## Congratulations!

You have completed the .NET tutorial. You are now ready to integrate all your .NET apps with Glue42 Enterprise!